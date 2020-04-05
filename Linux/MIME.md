# MIME
### What is MIME?
Handles default programs for filetypes (text/plain, text/x-c++, etc).
Theres a neato app that lets you graphically look at your 'default files' - [xfce4-mime-settings](https://www.archlinux.org/packages/extra/x86_64/xfce4-settings/) (part of the xfce4 settings app)app.
the arch page for this is [here](https://wiki.archlinux.org/index.php/default_applications)

The MIME config is usually in ```.config/mimeapps.list``` (older systems sometimes calls it defaults.list)

### Setting a default application
Example for setting a default file manager:
```xdg-mime default thunar.desktop inode/directory```\
Where inode/directory is the MIME type.

### What is XDG in relation to MIME
[xdg-utils](https://wiki.archlinux.org/index.php/Xdg-utils#xdg-open) is a set of tools that allows applications to easily integrate with the desktop environment of the user, regardless of the specific desktop environment that the user runs.
XDG uses MIME database to open files. ```xdg-open <filename.xyz>``` will open a file.
You can even do ```xdg-open .``` and it will open a file explorer at that location in the terminal.

XDG is part of LSB (linux standard base) which aims to make developing different distributions easier.

### Useful MIME types
Here are some good examples of MIME types that i have set up on my own system.
A list of types can also be found at this [website](https://www.freeformatter.com/mime-types-list.html).
```
[Default Applications]
inode/directory=thunar.desktop;
application/x-bittorrent=uget-gtk.desktop;
x-scheme-handler/magnet=uget-gtk.desktop
```

### Find the "name" of an application
All applications are stored in ```/usr/share/applications/*```.
