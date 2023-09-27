# Input

I'm a HUGE fan of ergonomics and love making my computer use easier where possible.

* Keyboard: WASD V2 87
* Primary mouse: Elecom Huge BT
* General use mouse: Logitech MX Master 3
* Gaming mouse: Razer deathadder 2013

### Display mouse buttons when pressed

#### Mouse only

```none
xev | awk -F'[ )]+' '/^ButtonRelease/ { a[NR+2] } NR in a { printf "%-3s %s\n", $5, $8 }'
```

#### Keyboard Only

```none
showkey -a
```

### Rebind a key

* install xdotools (execute virtual key events) and xbindkeys (rebind keys to do things when you press them)
You can check for Key IDs with either *xev* or *xbindkeys -k*
* You can see xdotools in action with an example command:
  1. `sleep 0.5 && xdotool click 2` Click the middle mouse button
  2. `sleep 0.5 && xdotool key 's'` Type the S key
* Run the config with `xbindkeys`

Here is an example of an xbindkeys config.

```bash
# /home/roland/.xbindkeysrc

# When you press a mouse button take a screenshot
"flameshot gui"
    m:0x0 + b:12   (mouse)

# When you press a mouse button press another mouse button
"sleep 0.2 && xdotool click 2"
    m:0x0 + b:10   (mouse)
```

### Scroll with the trackball

Set libinput Button Scrolling Button (283) to 9 *(9 is the mouse button number)*

```none
sudo xinput set-prop 12 283 9
```

Then set libinput Scroll Method Enabled (281) to 0, 0, 1

```none
sudo xinput set-prop 12 281 0, 0, 1
```

### Change mounse sensitivity

You can see the possible options using the list devices command. You can also use an [xinput graphical frontend](https://aur.archlinux.org/packages/xinput-gui/) to change the values .

```bash
xinput list-props "Razer Razer DeathAdder"
```

Heres a sample using xinput to change the sensitivity of a mouse.

```bash
xinput --set-prop "Razer Razer DeathAdder" "libinput Accel Speed" -0.95
xinput --set-prop "Razer Razer DeathAdder" "Coordinate Transformation Matrix" 1 0 0 0 1 0 0 0 1
xinput --set-prop "Razer Razer DeathAdder" "libinput Accel Profile Enabled" 0, 1
```

### Piper and ratbagd

ratbagd is an api that provides a way to customize different "gaming" or "special" features on different brands of mice. It works well with logitech mice especially (aparantly though not for me though).

To install on debian 10 you need to move to the "testing" branch instead of "buster", do this by editing `/etc/apt/sources.list` and changing "buster" to "testing".

```none
# replace "buster main" with "testing main".
deb http://dev.debian.org/debian testing main
deb-src http://dev.debian.org/debian testing main

# also change this to "testing" (not sure if required but works for me)
deb http://security.debian.org/debian-security testing main
deb-src http://security.debian.org/debian-security testing main

# leave this as is (this following block is default)
dev http://deb.debian.org/debian buster-updates main
dev-src http://deb.debian.org/debian buster-updates main
```

next run `sudo apt update` and then `sudo apt upgrade` to install these newer packages.

Next install piper with `sudo apt install piper`, piper will pull down ratbagd as its dependency. However if you just want to install ratbag WITHOUT piper as its frontend GUI, then installing `ratbag-tools` will work for that.

you can then run `piper` to configure any supported and conntected mice.

Some other CLI commands that may come in handy.

```output
# show the detected mice
ratbagctl list

# get help
ratbagctl info
```

### xmodmap

xmodmap is a tool used to remap buttons on keyboard, mice, and other input devices.

This tool has many uses, one use was to fix my mouse (logitech mx master 3) horizontal scroll wheel direction, i will explain the steps i used to do this and hopefully you can apply the general idea to any future problems.

Firstly i used the `xinput` tool to detect the device that i wanted to modify.

```
# list all the devices
xinput list

# then take a look at the props for this device
xinput list-props "pointer:Logitech MX Master 3"
xinput list-props "pointer:Logitech MX Master 3" | grep -i scroll
```
This is the output from the above command

```output
libinput Natural Scrolling Enabled (278):	0
libinput Natural Scrolling Enabled Default (279):	0
libinput Scroll Methods Available (280):	0, 0, 1
libinput Scroll Method Enabled (281):	0, 0, 0
libinput Scroll Method Enabled Default (282):	0, 0, 0
libinput Button Scrolling Button (283):	2
libinput Button Scrolling Button Default (284):	2
libinput Horizontal Scroll Enabled (295):	1
```

Unfortuantly the only options that are worth looking at is `libinput Natural Scrolling Enabled (278):	0`.
If we set option 278 using `sudo xinput set-prop "pointer:Logitech MX Master 3" 278 1` we WILL resolve the horizontal scroll, but break the vertical scroll wheel.
Luckily if your mouse only has one scroll wheel and you want to inverse the scroll direction this will fix it, however i had to keep looking because it didnt solve my problem.

Next i investigated using `piper` and `ratbagctl` however was also unable to fix the issue due to error messages caused while trying to rebind the mouse buttons, furthermore i was unsure which mouse buttons to set because piper failed to load the svg icon for my mouse.

The next step i tried was to use `xev` and find out if the mouse buttons were differentiated from each other (scrolling horizontally vs vertically on seperate scrollwheels would produce different keycodes), luckily this was the case.

Here is output using the horizontal scroll wheel

```output
ButtonRelease event, serial 34, synthetic NO, window 0x6c00001,
    root 0x421, subw 0x0, time 15637443, (162,78), root:(613,962),
    state 0x0, button 6, same_screen YES

ButtonPress event, serial 34, synthetic NO, window 0x6c00001,
    root 0x421, subw 0x0, time 15638079, (162,78), root:(613,962),
    state 0x0, button 7, same_screen YES
```

Here is output using the vertical scroll wheel

```output
ButtonRelease event, serial 34, synthetic NO, window 0x6c00001,
    root 0x421, subw 0x0, time 15686845, (60,74), root:(511,958),
    state 0x800, button 4, same_screen YES

ButtonPress event, serial 34, synthetic NO, window 0x6c00001,
    root 0x421, subw 0x0, time 15687297, (60,74), root:(511,958),
    state 0x0, button 5, same_screen YES
```

Using `xmodmap -pp` i can map my pointer devices buttons.

```output
There are 20 pointer buttons defined.

    Physical        Button
     Button          Code
        1              1
        2              2
        3              3
        4              4
        5              5
        6              7
        7              6
        8              8
        9              9
       10             10
       11             11
       12             12
       13             13
       14             14
       15             15
       16             16
       17             17
       18             18
       19             19
       20             20
```

Then using xmodmap i can swap the two buttons for horizontal scroll using the `-e` flag, Swapping the 6 and 7 button around to reverse the direction of the scroll.

```none
sudo xmodmap -e "pointer = 1 2 3 4 5 7 6 8 9 10 11 12 13 14 15 16 17 18 19 20"
```
