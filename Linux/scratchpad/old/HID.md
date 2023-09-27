<!-- https://who-t.blogspot.com/2018/12/understanding-hid-report-descriptors.html -->
# ConfigFS File System for Keyboard and Mouse Emulation

There are two ways to peek into what the kernel is seeing, **sysfs** and **configfs**.

**Sysfs** is a (mostly read only) filesystem that acts as a window to what is going on inside linux at the current time

**Configfs** is a (writeable) file system actually allows you to change data (persistently?)

You can find some more information about sysfs and configfs at this informational forum post [here](https://www.linux.org/threads/sysfs-and-configfs.9353/), and the kernel.org documentation of configfs [here](https://www.kernel.org/doc/Documentation/filesystems/configfs/configfs.txt).

To load configfs when we boot up for our project we need to modify `/etc/modules` to load in modules that use configfs.

* Loading libcomposite causes /sys/kernel/config/usb_gadget to appear
* Loading dwc2 and adding `dtoverlay=dwc2` to `/boot/config.txt` just allows it to flip to a slave device

```none
# /etc/modules: kernel modules to load at boot time.
#
# This file contains the names of kernel modules that should be loaded
# at boot time, one per line. Lines beginning with "#" are ignored.

# dwc2 is an upstream driver which can do the OTG host/gadget flip
dwc2

# libcomposite provides configfs
libcomposite
```

We also need to tell the PI to act like a slave controller, add this to `/boot/config.txt` to modify the Device Tree (DT) which describes the hardware present on the PI. See [this documentation from raspberry](https://www.raspberrypi.org/documentation/configuration/device-tree.md) for more information on device trees and device tree overlays. We are actually enabling an entire overlay which enables an entire list of device dependencies that we don't need to worry about.

```none
# /boot/config.txt

[all]
#dtoverlay=vc4-fkms-v3d
dtoverlay=dwc2
```

## Report descriptors

A report descriptor describes what a HID device is capable of.

Get the location of your mouse

```none
sudo file /dev/input/by-id/usb-Logitech_USB_Receiver-if02-mouse
```

```output
/dev/input/by-id/usb-Logitech_USB_Receiver-if02-mouse: symbolic link to ../mouse2
```

Then decode the report_descriptor using hid-tools.

```none
su -
pip3 install hid-tools

hid-decode /sys/class/input/mouse2/device/device/report_descriptor
```

## Splitting up Report Descriptors

Lets take the following report descriptor and split it up using these rules.

* lines `0 - 4` are always the same (at least for a keyboard)
* A `Usage Page (anything here)` **SOMETIMES** marks the beginning of a block
* An `Input` and `Output` **ALWAYS** (unless its at the beginning or end) marks the end of a block

A **block** is literally a block of information, usually grouped as a byte (8 bits). Below is a keyboard split up correctly.

```none
// lets call this block 1
// Just tells us generic information, that this is a computer keyboard.
x05, x01,                    // Usage Page (Generic Desktop)        0
x09, x06,                    // Usage (Keyboard)                    2
xa1, x01,                    // Collection (Application)            4

// lets call this block 2
// We know this is mod keys because Usage Minimum is 224 (see below for further)
x05, x07,                    //  Usage Page (Keyboard)              6
x19, xe0,                    //  Usage Minimum (224)                8
x29, xe7,                    //  Usage Maximum (231)                10
x15, x00,                    //  Logical Minimum (0)                12
x25, x01,                    //  Logical Maximum (1)                14
x75, x01,                    //  Report Size (1)                    16
x95, x08,                    //  Report Count (8)                   18
x81, x02,                    //  Input (Data,Var,Abs)               20

// lets call this block 3
// almost always a spare byte
x95, x01,                    //  Report Count (1)                   22
x75, x08,                    //  Report Size (8)                    24
x81, x01,                    //  Input (Cnst,Arr,Abs)               26

// lets call this block 4
// This is for LEDs because it says so
x95, x03,                    //  Report Count (3)                   28
x75, x01,                    //  Report Size (1)                    30
x05, x08,                    //  Usage Page (LEDs)                  32
x19, x01,                    //  Usage Minimum (1)                  34
x29, x03,                    //  Usage Maximum (3)                  36
x91, x02,                    //  Output (Data,Var,Abs)              38

// lets call this block 5
// LEDs was only 3 bits long, so this is a 5 bit empty block to make up the 1 byte
x95, x05,                    //  Report Count (5)                   40
x75, x01,                    //  Report Size (1)                    42
x91, x01,                    //  Output (Cnst,Arr,Abs)              44

// lets call this block 6
// this is the keys because its large (6x8 bits, 6 bytes) to support multiple keys
x95, x06,                    //  Report Count (6)                   46
x75, x08,                    //  Report Size (8)                    48
x26, xff, x00,              //  Logical Maximum (255)               50
x05, x07,                    //  Usage Page (Keyboard)              53
x19, x00,                    //  Usage Minimum (0)                  55
x29, x91,                    //  Usage Maximum (145)                57
x81, x00,                    //  Input (Data,Arr,Abs)               59

xc0,                          // End Collection                      61
```

**Block 2** - We know this is likely **modifier keys** because the **Usage Minimum** is 224. This means that when we send `0b10 (2)` in this block we actually sent `(0d224 + 0b10) - 1` or `11100000 + 00000010 - 00000001 = 011100001` which is usage ID 225 which on the HID Usage Table [here](https://www.usb.org/sites/default/files/documents/hut1_12v2.pdf) (pg53) is decoded to be a keyboard Left Shift. This is described below in "Determine Modifier Keys".

## Counting the report length

When creating a "packet" to send down the wire we need to know the report length which determines the size of the packet we are sending. We can get this information from the report descriptor.

**REMEMBER** to discount any block that ends in `output` because output is sent from the master PC (not us) to the slave PI (us) so we dont count this in any data WE are going to send.

* Block 1 - skip
* Block 2 - (8 bits) this is `Report Size (1) * Report Count (8)` so its 8 bits or 1 byte
* Block 3 - (8 bits) Using above, so its 8 bits  (this is the spare byte block)
* Block 4 - (3 bits) This is just 3 bits (see next block, 5)
* Block 5 - (5 bits) This is just 5 bits (makes up the missing bits in block 4) **we will report the 3 + 5 bit block as ONE byte**
* Block 6 - (48 bits) This is 6*8 bits so its 6 bytes

So `8 + 8 + 3 + 5 + 48 = 72 bits, or 9 bytes` is the length of the report descriptor (read on).
**BUT** we need to remove the output blocks so lets **subtract** the 3 and 5 bit blocks (-8 bits) to make the **final value  of 8 bytes**.

TLDR Remember to not count output blocks...

### Logical Minimum and Logical Maximum

The Logical Minimum and Maximum represent the minimum and maximum of the actual data. Think of it as a clamp on what data can be sent. THe reason why we include this is to let the host know if we are sending signed bits or not.

If the data we are sending has a Logical Minimum of < 1 then it will be signed. If the Min and Max are both > 1 then its unsigned.

Lets look at an example.

```none
Logical Minimum (0)
Logical Maximum (1)
Physical Minimum (1)
Physical Maximum (12)
```

* If we sent x00 we will actually send 1 (because the physical minimum is 1)
* If we sent x01 we will actually send 12 (because the physical minimum is )

However we typically don't use Physical Min and Max so heres a simpler example without. We don't need to include the Physical Min and Max if the data we are sending is exactly the same range as the logical Min and Max.

```none
// in this example we don't need a physical min and Max
// we send 1 byte
// that byte can contain any value between 0 and 255

Report Count (1)
Report Size (8)
Logical Minimum (0)
Logical Maximum (255)

// in this example we need a min and max
// we need to send data between 224 and 231
// also we are wasting bits here but this is just for example
Report Count (1)
Report Size (8)
Logical Minimum (0)
Logical Maximum (255)
Physical Minimum (224)      // sending x00 is parsed as x224
Physical Maximum (231)      // sending xff (0d255) is parsed as x224
```

### Usage Minimum and Usage Maximum

The Usage Minimum and Usage Maximum exist to order a block sequentially. Take a look at the example.

```none
Usage Minimum (1)       // a range between 1 and 5 (imagine a x=0; x<5 loop)
Usage Maximum (5)
Report Count (5)        // the total block will be 5 bits long
Report Size (1)         // in 1 bit chunks
Input (Data,Var,Abs)
```

We have 5 buttons, 1, 2, 3, 4, 5. pressing one button could look like `00001` or two buttons `01001`, the Usage Min and Max essentially duplicate this block into 5 blocks, the first two of which would look like this.

```none
// replace Usage with Logical
// Instead of a Usage Block with 1 to 5
// We replace this with the Logical of 1 to 1 because its just 1 bit of the range (that usage provided us with)
Logical Minimum (0)     // this bit can be either
Logical Maximum (1)     // zero or one
Physical Minimum (0)    // sending 0 is parsed as x00
Physical Maximum (1)    // sending 1 is parsed as x01
Report Count (1)        // just 1 report
Report Size (1)         // the report is 1 bit long
Input (Data,Var,Abs)

// the second one
Logical Minimum (0)     // this bit can be either
Logical Maximum (1)     // zero or one
Physical Minimum (0)    // sending x00 is parsed as x00
Physical Maximum (2)    // sending x01 is parsed as x02     < -- we only changed this value
Report Count (1)        // just 1 report
Report Size (1)         // the report is 1 bit long
Input (Data,Var,Abs)

// the third one etc
Logical Minimum (0)     // this bit can be either
Logical Maximum (1)     // zero or one
Physical Minimum (0)    // sending x00 is parsed as x00
Physical Maximum (3)    // sending x01 is parsed as x03     < -- we only changed this value
Report Count (1)        // just 1 report
Report Size (1)         // the report is 1 bit long
Input (Data,Var,Abs)
```

In a more real world example lets look at another block that describes modifier keys.

```none
// mod keys
Usage Page (Key Codes)
Report Size (1)        // 1 bit
Report Count (8)       // repeated 8 times
Usage Minimum (224)    // between 224
Usage Maximum (231)    // and 231
Logical Minimum (0)    // each bit is between 0 and 1
Logical Maximum (1)
```

We can send up to 8 mod keys at a time (Report Count (8)). Example `00000001` presses 1 key (224) and `00000101 or x05` Presses 2 keys (224 and 226). Instead of using the Usage Max and Min to create a range, instead we can write the first bit as a single block.

```none
Usage Page (Key Codes)
Report Size (1)
Report Count (1)
// Instead of a Usage Block with 1 to 5
// We replace this with Logical 0 to 1 because its just one one bit
Logical Minimum (0)
Logical Maximum (1)
```

## Determine Modifier Keys

To understand the usage minimum for a mod key have a look at this following section from a report descriptor that describes the mod keys.

```none
x05, x01, // Usage Page (Usage Page (Generic Desktop)
x09, x06, // Usage (Keyboard)
xa1, x01, // Collection (Application)

// -- MODIFIER KEYS --
x05, x07, // Usage Page (Key Codes) (this tell the host that this field block is used for keys)
x75, x01, // Report Size (1) (each modifier is only 1 bit)
x95, x08, // Report Count (8) (there are 8 modifiers)
x19, xe0, // Usage Minimum (224) (this field block represents the keys in the range 224-231 (modifiers))
x29, xe7, // Usage Maximum (231) (this field block represents the keys in the range 224-231 (modifiers))
x15, x00, // Logical Minimum (0) (the fields in this block can only be 0 or 1)
x25, x01, // Logical Maximum (1) (the fields in this block can only be 0 or 1) so you can be values of either 0b01 or 0b00
x81, x02, // Input (Data, Variable, Absolute ()) (input means slave device -> host device)
            //                           ^ absolute means the absolute state of the modifiers are being reported.
            //                             this is not the case for the field block down the bottom for normal keys
            //                             absolute = slave tracks it (or doesn't really need to be tracked at all)
```

We need the `Usage Minimum (224)` to be set to 224 for modifier keys because mod keys happen to begin at Usage ID 224 for the **Keyboard/Keypad Page (x07)**.

To determine the mod key to be sent, use this formula.

```none
1 << (usage_code - usage_minimum)
```

Using the HID Usage Table [here](https://www.usb.org/sites/default/files/documents/hut1_12v2.pdf) (pg53) we can determine that a Left Super, also known as `Keyboard Left GUI` is Usage ID 227 (in decimal) and its Hex is `xe3`. So lets sub that in for a usage minimum of 224 which is set for modifier keys.

```none
0b1 << (227 - 224)
```

```none
0b1 << 3
```

```none
1 = 0001

// add 3 zeros to the right
0b1 << 3 = 1000
```

now that its shifter we have a full "shippable packet" of `00001000`.

Now lets add another modifier key to send (imagine pressing two keys at once, each bit in the byte represents one key).

Lets also send a `Left Mod` or 224, so that gives us a value of 0d1, or `00000001`

To send both packets lets combine them together through a binary OR operation.

```none
00001000
00000001
---
00001001
```

So to send a **Left Control** and **Left Mod** pressed simultaneously you will send `00001001` in the **Modifier Keys** section of the report descriptor.

## Sending data

Now that we know the blocks and the report length, we can send a packet. We need 8 bytes in total (our report length = 8).

Lets say we want to press the "a" key.

<!-- * Byte 1 - 0x00 don't worry we always (at the moment) send 00 for the first byte (block 1) -->
* Byte 1 - 0x00 we wont press any modifier keys
* Notice that we omitted the LED block because its output, keep going
* Byte 2 - 0x00 spare block
* Byte 3 - 0x04 key 1 <- press the a key
* Byte 4 - 0x00 key 2
* Byte 5 - 0x00 key 3
* Byte 6 - 0x00 key 4
* Byte 7 - 0x00 key 5
* Byte 8 - 0x00 key 6

Lets send that as a "packet" to our device `hidg0`.

```none
echo -ne "\x00\x00\x04\x00\x00\x00\x00\x00" > /dev/hidg0
```

## Understanding the Mouse Report Descriptor

Lets use a real life example for my Razer RGB gaming mouse. I have already split up and commented the report descriptor using the notes above.

```none
0x05, 0x01,                    // Usage Page (Generic Desktop)        0
0x09, 0x02,                    // Usage (Mouse)                       2
0xa1, 0x01,                    // Collection (Application)            4
0x09, 0x01,                    //  Usage (Pointer)                    6
0xa1, 0x00,                    //  Collection (Physical)              8

// Mouse clicks
0x05, 0x09,                    //   Usage Page (Button)               10
0x19, 0x01,                    //   Usage Minimum (1)                 12
0x29, 0x07,                    //   Usage Maximum (7)                 14
0x15, 0x00,                    //   Logical Minimum (0)               16
0x25, 0x01,                    //   Logical Maximum (1)               18
0x95, 0x08,                    //   Report Count (8)                  20
0x75, 0x01,                    //   Report Size (1)                   22
0x81, 0x02,                    //   Input (Data,Var,Abs)              24

// Its hard to determine without using the hid-recorder tool and tinkering (its a vendor defined thing)
0x06, 0x00, 0xff,              //   Usage Page (Vendor Defined Page 1) 26
0x09, 0x40,                    //   Usage (Vendor Usage 0x40)         29
0x95, 0x02,                    //   Report Count (2)                  31
0x75, 0x08,                    //   Report Size (8)                   33
0x15, 0x81,                    //   Logical Minimum (-127)            35
0x25, 0x7f,                    //   Logical Maximum (127)             37
0x81, 0x02,                    //   Input (Data,Var,Abs)              39

// The scroll wheel
0x05, 0x01,                    //   Usage Page (Generic Desktop)      41
0x09, 0x38,                    //   Usage (Wheel)                     43
0x15, 0x81,                    //   Logical Minimum (-127)            45
0x25, 0x7f,                    //   Logical Maximum (127)             47
0x75, 0x08,                    //   Report Size (8)                   49
0x95, 0x01,                    //   Report Count (1)                  51
0x81, 0x06,                    //   Input (Data,Var,Rel)              53


// The X and Y position as a signed 16 bit integer
0x09, 0x30,                    //   Usage (X)                         55
0x09, 0x31,                    //   Usage (Y)                         57
0x16, 0x00, 0x80,              //   Logical Minimum (-32768)          59
0x26, 0xff, 0x7f,              //   Logical Maximum (32767)           62
0x75, 0x10,                    //   Report Size (16)                  65
0x95, 0x02,                    //   Report Count (2)                  67
0x81, 0x06,                    //   Input (Data,Var,Rel)              69

0xc0,                          //  End Collection                     71
0xc0,                          // End Collection                      72
```

Next lets determine the report length using the blocks we have outlined.

Report Length = `8 + 16 + 8 + 32` = 64 bits = 8 bytes.

And while we are at it lets create the report descriptor in HID readable format

```none
\\x05\\x01\\x09\\x02\\xa1\\x01\\x09\\x01\\xa1\\x00\\x05\\x09\\x19\\x01\\x29\\x07\\x15\\x00\\x25\\x01\\x95\\x08\\x75\\x01\\x81\\x02\\x06\\x00\\x09\\x40\\x95\\x02\\x75\\x08\\x15\\x81\\x25\\x7f\\x81\\x02\\x05\\x01\\x09\\x38\\x15\\x81\\x25\\x7f\\x75\\x08\\x95\\x01\\x81\\x06\\x09\\x30\\x09\\x31\\x16\\x00\\x80\\x26\\xff\\x7f\\x75\\x10\\x95\\x02\\x81\\x06\\xc0\\xc0
```

So just the the keyboard we know that an example packet will be 8 bytes in length (thats the length of the report descriptor).

```none
mouse clicks
|    vendor thing
|    |   Scroll Wheel
|    |   |   spare byte
|    |   |    |   |-x-|   |-y-|
\x00\x00\x00\x00\x00\x00\x00\x00
```

### Moving the mouse

So when we want to send some data, for example moving the mouse 64 units down and up, we need to do some math and learn some more things.

To do this example we will work off a simpler 8x2 report size (1 byte per axis instead of 2).

```none
Usage Page (Generic Desktop)
Usage (X)
Usage (Y)
// the value 127 here is chosen because we have 7 bits to work with and so 01111111 = 127
Logical Minimum (-127) // this is negative so that means that the right most bit is signed
Logical Maximum (127) // the report size is 7 (8 - 1 sign bit) bits so this is reflecting that
Report Size (8)
Report Count (2)
Input (Data,Variable,Relative)
```

Note how we have a Logical Minimum and Logical Maximum that is negative and positive, this indicates that we will need a signed bit. The signed bit will be the least significant bit (right most) and a 0 signed bit will indicate **down**, and a 1 will indicate **up**.

Lets say we want to move 64 units down. The working for this would be like follows.

```none
64 = 01000000

// convert to hex
01000000 = x40
```

Notice how when we are moving down, the bit of least value (on the right) does not change. we will ensure this bit is 1 when we want to move the mouse up.

An additional step for moving the mouse up is performing a twos compliment operation (invert) on the byte. For example to move up 64 units.

```none
64 = 01000000

// convert to hex
01000000 = x40

// twos compliment
10111111 = xBF

// ensure that the end is a 1 (thats the signed bit)
1011111(1) <- yep thats a 1
```

So in summary. To move **down** `x00 x40` and to move **up** `x00 xBF`.

**DISCLAIMER** Just make sure you are aware that when i say 64 units, i don't mean 64 pixels. Typically in testing i found that small units <= 16 pixels result in mouse acceleration (probably) taking over and messing with the numbers, whereas bug numbers >= 64 tend to go through more predictably.

<!-- So when we want to send some data, for example move the mouse down 10 units (unit is not px and will be interpreted by the OS based on mouse sensitivity). We can do this with the last two bytes (16 bits), however we need to convert the little endian to big endian first to make this work and actually make this understandable for humans.

So lets say we want to move (teleport) the mouse 256 units down (roughly 250 pixels but its not guaranteed because the OS mouse sensitivity will alter the final number).

First lets represent the last two bytes as binary, and add 1 to the end (1 will become 256 after the endian swap).

```none
00000000 00000001
```

Now lets swap the endian around from little endian to big endian.

```none
32768                       256     128                         1
0   0   0   0   0   0   0   0   |   0   0   0   0   0   0   0   1 <--- the 1 we added
──────────────┬──────────────       ────────────┬────────────────
              │                                 │
              └────────────────────────────┐    │
                                           │    │
                ┌──────────────────────────┼────┘
                │                          │
                │                          │
                ▼                          ▼
─────────────────────────────       ─────────────────────────────
0   0   0   0   0   0   0   1   |   0   0   0   0   0   0   0   0
                            |
                            now the byte with the 1 is swapped
```

Now our last 2 byte input of `00000000 00000001` or `0x00 0x01`.
When it is sent to the master device will actually be interpreted as `00000001 00000000` or `x01 x00` and if you look at the timeline the value `x01 x00` which is 16 bits will have the value of 256.

```none
       is now big endian 256
       |
00000001 00000000
                |
        the little endian 1
```

So lets put that into our report "packet" the way we first intended, however now that we know the value is going to be swapped we have knowledge that `x00 x01` is not sending 1 unit of movement, but actually 256 units of movement.

```none
\x00\x00\x00\x00\x00\x00\x00\x01
``` -->

## Base Config

As part of a project i want to emulate a keyboard and mouse which requires interaction with configfs to configure a raspberry pi zero as a wireless usb slave.

In [this](http://www.isticktoit.net/?p=1383) tutorial by isticktoit, the maker uses the `/sys/kernel/config/usb_gadget/` directory to modify the configfs file system to add a new usb HID device. I was able to comment and understand these sections through the documentation [here](https://www.kernel.org/doc/html/v4.16/driver-api/usb/gadget.html) and [here](https://android.googlesource.com/kernel/msm/+/android-msm-marlin-3.18-nougat-dr1/Documentation/usb/gadget_configfs.txt) that describes this boilerplate.

```none
#!/bin/bash

# File: /usr/bin/isticktoit_usb
# Feel free to change serial number, manufacturer and product name in this block.

# Go to configfs
cd /sys/kernel/config/usb_gadget/

# Create a new usb gadget
mkdir -p isticktoit
cd isticktoit

# Configure the new usb gadget
echo x1d6b > idVendor # Linux Foundation
echo x0104 > idProduct # Multifunction Composite Gadget
echo x0100 > bcdDevice # v1.0.0
echo x0200 > bcdUSB # USB2

# x0409 for en-us
mkdir -p strings/x409

# serial number, manufacturer, and product name
echo "fedcba9876543210" > strings/x409/serialnumber
echo "Tobias Girstmair" > strings/x409/manufacturer
echo "iSticktoit.net USB Device" > strings/x409/product

# mkdir configs/<name>.<number>
# <name> can be any string
# <number> is the configuration (1 is the first configuration)
mkdir -p configs/c.1/strings/x409

# I think this is for Ethernet over USB
echo "Config 1: ECM network" > configs/c.1/strings/x409/configuration

# Power consumption in mA.
echo 250 > configs/c.1/MaxPower

# Add functions here
# =====================================
# See the next section Adding Functions
# =====================================
# End functions

# Enable the usb_gadget
# In order to enable the gadget, it must be bound to a UDC (USB Device Controller)
# When libcomposite and dwc2 is setup there will be a file called xxxx.usb in /sys/class/udc
# /sys/class/udc/xxxx.usb must be linked to /sys/kernel/config/usb_gadget/isticktoit/UDC
ls /sys/class/udc > UDC
```

## Adding functions

USB Gadgets provide functions that are going to be located in `/sys/kernel/config/usb_gadget/isticktoit/functions`.

For each function its corresponding directory must be created.

```none
mkdir /sys/kernel/config/usb_gadget/isticktoit/functions/<name>/<instance name>
```

For the keyboard...

```none
# Create the function directory like above
mkdir -p functions/hid.usb0

# Speak the magic words to be a keyboard
echo 0 > functions/hid.usb0/protocol        # just make this a 0 idk why yet
echo 0 > functions/hid.usb0/subclass        # just make this a 0 idk why yet
echo 8 > functions/hid.usb0/report_length   # We discussed report length above

# This is the actual report descriptor with all the comments removed
echo -ne \\x05\\x01\\x09\\x06\\xa1\\x01\\x05\\x07\\x19\\xe0\\x29\\xe7\\x15\\x00\\x25\\x01\\x75\\x01\\x95\\x08\\x81\\x02\\x95\\x01\\x75\\x08\\x81\\x03\\x95\\x05\\x75\\x01\\x05\\x08\\x19\\x01\\x29\\x05\\x91\\x02\\x95\\x01\\x75\\x03\\x91\\x03\\x95\\x06\\x75\\x08\\x15\\x00\\x25\\x65\\x05\\x07\\x19\\x00\\x29\\x65\\x81\\x00\\xc0 > functions/hid.usb0/report_desc

# Associate the configuration created in the base config with the function
ln -s functions/hid.usb0 configs/c.1/
```
