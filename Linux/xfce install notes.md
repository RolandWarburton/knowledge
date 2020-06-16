# xfce install notes

1. Install xfce

```none
sudo pacman -S xfce4 xfce4-settings
```

2. Add xfce to start at boot in xinitrc

```none
exec startxfce3
```

3. Add keyboard shortcut to dmenu

```none
Applications -> settings -> keyboard -> Application Shortcuts -> "dmenu_run" bound to ALT+SPACE
```

4. Change the taskbar panel options
   1. Run xfce4-settings-manager (if you have removed the 'Applications' launcher from the taskbar already)
   2. navigate to panel and change settings to how you like
   3. Change the panel to use the system style or change its background color
   4. Configure window buttons: Change window buttons to 'show flat buttons' and sort by 'window title' and un-check 'show button labels'.
   5. Configure clock: use custom date format %I:%M
5. Get the theme working
   1. use 'xfce4-appearance-settings' and select a theme
   2. good themes are arc-dark
6. Get lazy tiling by going to 'xfce4-settings-manager -> window manager -> keyboard -> Tile window to the X'. Also remember to change the window snapping under 'advanced' to snap to screen borders and make the distance a good number to prevent windows from being moved to the next virtual desktop
7. Yeet the window borders from your xfce by creating an empty xfce window manager theme. Go to 'xfce4-settings-manager -> window manager -> style' and select the 'empty' theme. Its literally a theme with no styles in it. So you will get no borders or anything.

```none
sudo mkdir -p /usr/share/themes/empty/xfwm4/
sudo touch /usr/share/themes/empty/xfwm4/themerc
```

8. Get the screenshot tool working with 'xfce4-settings-manager -> keyboard' and binding the 'Print' key (printscreen) to your screenshot program (flameshot is my preferred program)
9. Get moving windows to move to next/prev screen with wmctrl
10. Fix an issue in VSCode where XFCE takes over the ALT key and you cannot alt+up/down lines of code by rebinding the alt key in VSC to MOD (or something else). press f1 and type f1 and search for 'Preferences: Open Keyboard Shortcuts'.
