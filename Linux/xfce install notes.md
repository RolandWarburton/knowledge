# xfce install notes


1. Install xfce
```sudo pacman -S xfce4 xfce4-settings```
2. Add xfce to start at boot in xinitrc
```exec startxfce3```
3. Add keyboard shortcut to dmenu
```Applications -> settings -> keyboard -> Application Shortcuts -> "dmenu_run" bound to ALT+SPACE```
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