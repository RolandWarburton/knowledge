
# .xinitrc, .xsession, and .xresources

### .xinitrc

Runs when you run the startx command. This should be autorun when you log in to start your DE or WM by adding a line to zshrc or bashrc to call it as soon as the shell starts.

```none
# autostart xinit (startx) on your shell (same on zsh and bash etc)
if systemctl -q is-active graphical.target && [[ ! $DISPLAY && $XDG_VTNR -eq 1 ]]; then
  exec startx
fi
```

### startx vs xinit

xinit is a more bare bones version of startx. xinit only starts the X server and runs anything in the users xinitrc. you will get a black screen and no DE or WM if nothing is specified in your xinitrc.

startx starts the X server and looks for additional files. Have a look at `which startx` to see the  startx script.

### .xsession

xsession is used by graphical login managers like sddm and gdm and is seperate from xinitrc. (xinitrc is used by startx, xorg and init when starting an actual DE or WM).
Though xsession can be used as a fallback if no .xinitrc is found.

Runs when a graphical environment starts. Its purpose is to contain user configuration and may contain

* Environment variables. `export VARIABLE=thing`
* Calling other scripts to run `if [ -r ~/.runascript ]; then . ~/.runascript; fi`
* starting programs `chromium`

Though it is considered *advanced* (and therefore had no great documentation), xsession may also start your DE or WM like how you would run it in xinitrc. This avoids the need for a .xinitrc and using the *startx* command which is deprecated on some distros (openSUSE).

```none
# .xsession
exec dbus-launch openbox-session
```

### .xresources

Has to be called by *.xinitrc* or *.xsession* to run. It is part of the xorg-xrdg package and configures parameters (usually appearance) for X client applications (like xterm and xclock etc).
xinitrc by default merges (combines the xresources with current config so you dont have to type all the things you dont want to change again). You can also technically load xresources from *.xsession*.

The default settings that .xresources merges with or overrides are at `/usr/share/X11/app-defaults/`

Command to merge xresources for either .xinitrc (prefered) or .xsession: `[[ -f ~/.Xresources ]] && xrdb -merge -I$HOME ~/.Xresources`

You can also check currently loaded resources to confirm if anything was loaded with `xrdb -query -all`
