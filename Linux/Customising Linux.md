# Customizing linux

Change the settings on your linux install to set up the staple parts of your install.

This encompasses both fixing bugs in common workflows, setting up minimal themes, and other small QOL changes.

### Set the background color like a real hackerman

Dont use any programs to change your background color. When using an X display server you can simply append `xsetroot -solid "#121212"` to your xinitrc.
Another way to do this is `exec --no-startup-id xsetroot -solid "#121212"`

### Style QT applications like GTK ones

I learnt how to use GTK themes before QT ones. So i needed a way to use my preferred GTK theme to style everything. This is done through a *Theme Engine* which is a non-elegant solution to translate GTK styles to QT styles. QT has such an engine called `QGtkStyle`. However it was depricated (removed) in QT5. Install [qt5-styleplugins](https://www.archlinux.org/packages/community/x86_64/qt5-styleplugins/) to use QT's theme engine in modern versions of QT.

> Set the environment variable `$QT_STYLE_OVERRIDE=gtk2`. This will work for the Arc-Dark theme i am using right now

### Set a desktop background (image) like a real hackerman

This can be frustrating to figure out on some desktop environments (like openbox) where the background is set in a config file hidden somewhere.
In Openbox's case you need to modify `vim /usr/lib/openbox/openbox-autostart` and comment out the code block from `#Set a background color`.

The lightest weight method (that supports dual screen) is to use [feh](https://www.archlinux.org/packages/extra/x86_64/feh/).
Modify your `~/.xinitrc` to contain `exec feh --bg-fill /home/roland/Media/Backgrounds/Catlesstail_portrait.png /home/roland/Media/Backgrounds Catlesstail.png &`. Optionally uou can pass seperate image paths to set multiple monitors.

### Get all the emojis

* Install the correct packages for emojis `sudo pacman -S noto-fonts-emoji`.
* Then modify either `/etc/fonts/local.conf` for global font config or `~/.config/fontconfig/fonts.conf` for per user config.
* Create a config file to favour the font emoji.

```xml
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
 <alias>
   <family>sans-serif</family>
   <prefer>
     <family>Noto Sans</family>
     <family>Noto Color Emoji</family>
     <family>Noto Emoji</family>
     <family>DejaVu Sans</family>
   </prefer>
 </alias>

 <alias>
   <family>serif</family>
   <prefer>
     <family>Noto Serif</family>
     <family>Noto Color Emoji</family>
     <family>Noto Emoji</family>
     <family>DejaVu Serif</family>
   </prefer>
 </alias>

 <alias>
  <family>monospace</family>
  <prefer>
    <family>Noto Mono</family>
    <family>Noto Color Emoji</family>
    <family>Noto Emoji</family>
   </prefer>
 </alias>
</fontconfig>
```

### Moving Zsh config files to nicer locations

Configure the global variable $ZDOTDIR to change where zsh looks for its config files.

```bash
/etc/zsh/zshenv
export ZDOTDIR=$HOME/.config/zsh
```

### Making thunar dark theme

Thunar is a GTK (gimp toolkit) app. This is a generic library used for graphics interfaces.
Gtk can be themed, and therefore will incluence all the GTK widgets that make up these applications.
You can configure thunar and the system wide theme with lxappearance.\
lxappearance is part of Xappearance: *XAppearance — Desktop independent GTK 2 and GTK 3 style configuration tool from the LXDE project (it does not require other parts of the LXDE desktop).*

1. install lxappearance
2. run and select a theme!

#### Make thunar dark theme on xfce4

run `xfce4-appearance-settings` and change the GTK theme through there

### Install an icon theme

The best way i found to do this is through lxappearance.\
Icons should be in the `/usr/share/icons` folder for all users, or the `~/icons/` folder for local users. Avoid putting icon themes in the home directory.

1. Install an icon theme through the repos or AUR
2. use the icon-theme tab on lxappearance to select that theme, it should appear there automatically

### What is the best terminal font

TTF-Hack 😄

Also on the topic of fonts you should install FiraCode for font ligatures. Instructions [here](https://github.com/tonsky/FiraCode/wiki/VS-Code-Instructions)
