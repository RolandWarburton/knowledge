# Input
I'm a HUGE fan of ergonomics and love making my computer use easier where possible.
* Keyboard: WASD V2 87
* Primary mouse: Elecom Huge BT
* General use mouse: Logitech MX Master 3
* Gaming mouse: Razer deathadder 2013

### Display mouse buttons when pressed
```xev | awk -F'[ )]+' '/^ButtonRelease/ { a[NR+2] } NR in a { printf "%-3s %s\n", $5, $8 }'```

### Rebind a key
* install xdotools (execute virtual key events) and xbindkeys (rebind keys to do things when you press them)
You can check for Key IDs with either *xev* or *xbindkeys -k*
* You can see xdotools in action with an example command: 
1. ```sleep 0.5 && xdotool click 2``` Click the middle mouse button
2. ```sleep 0.5 && xdotool key 's'``` Type the S key
* Here is an example of an xbindkeys config
```
# /home/roland/.xbindkeysrc

# When you press a mouse button take a screenshot
"flameshot gui"
    m:0x0 + b:12   (mouse)

# When you press a mouse button press anopther mouse button
"sleep 0.2 && xdotool click 2"
    m:0x0 + b:10   (mouse)
```
* Run the config with ```xbindkeys```

### Scroll with the trackball
Set libinput Button Scrolling Button (283) to 9 *(9 is the mouse button number)*
```
sudo xinput set-prop 12 283 9
```

Then set libinput Scroll Method Enabled (281) to 0, 0, 1
```
sudo xinput set-prop 12 281 0, 0, 1
```
