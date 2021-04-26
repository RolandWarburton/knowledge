# Macro Keyboard With actkbd

cat /proc/bus/input/devices | grep "Chroma" -B 1 -A 9

Looking for `H` -> `H: Handlers=sysrq kbd leds event25`.

```none
I: Bus=0003 Vendor=1532 Product=0203 Version=0111
N: Name="Razer Razer BlackWidow Chroma"
P: Phys=usb-0000:03:00.0-1.1/input0
S: Sysfs=/devices/pci0000:00/0000:00:1c.4/0000:03:00.0/usb5/5-1/5-1.1/5-1.1:1.0/0003:1532:0203.000C/input/input33
U: Uniq=
H: Handlers=sysrq kbd leds event25
B: PROP=0
B: EV=120013
B: KEY=1000000000007 ff9f207ac14057ff febeffdfffefffff fffffffffffffffe
B: MSC=10
B: LED=7
```

Then listen for key presses.

```none
sudo actkbd -s -d /dev/input/event25
```

```output
❯ sudo actkbd -s -d /dev/input/event25
Keys: 30
Keys: 31
Keys: 32
```

Then we need the key board daemon to intercept these events

```none
git clone https://github.com/thkala/actkbd.git
cd actkbd
```

Then run these.

```none
sudo make -n install
sudo make install
```

actkbd should now be installed. You can modify `/etc/actkbd.conf` now for a test program.
For example when you press the `a` key run VSC.

```none
# actkbd configuration file

# actkbd is run as root so we need to run code as the user
30:::sudo -u roland /usr/bin/code
```

Then all you need to do is start the daemon.

```none
sudo actkbd -d /dev/input/event25
```
