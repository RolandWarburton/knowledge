# Fonts

System fonts can be found in `/usr/share/fonts/`.

```none
ls -l /usr/share/fonts
```

```output
total 24K
drwxr-xr-x  2 root root 4.0K Nov 30 15:58 cmap
drwxr-xr-x  2 root root 4.0K Nov 30 15:58 cMap
drwxr-xr-x  5 root root 4.0K Nov 30 18:25 opentype
drwxr-xr-x 14 root root 4.0K Dec  2 10:06 truetype
drwxr-xr-x  4 root root 4.0K Nov 30 18:25 type1
drwxr-xr-x  8 root root 4.0K Nov 30 16:00 X11
```

We only care about `opentype` and `truetype` really, each corresponds to otf and ttf respectively.

When we install new fonts, they are installed to either the opentype (otf), or truetype (ttf) directory.

## Installing a Custom Font

This can be done by downloading the font, and unzipping it. Then just moving the files to their correct destination.

For example when installing a new font (firecode nerd font).

```none
# usually run as root
mkdir /usr/share/fonts/truetype/firacode-nerd-font
mkdir /usr/share/fonts/opentype/firacode-nerd-font

# usually run as root
mv *.ttf /usr/share/fonts/truetype/firacode-nerd-font
mv *.otf /usr/share/fonts/opentype/firacode-nerd-font

# always run as root
sudo chown root:root /usr/share/fonts/truetype/firacode-nerd-font -R
sudo chown root:root /usr/share/fonts/opentype/firacode-nerd-font -R

# always run as root
sudo chmod 664 /usr/share/fonts/truetype/firacode-nerd-font/*
sudo chmod 664 /usr/share/fonts/opentype/firacode-nerd-font/*
```

After installing, run `fc-cache` to update the font cache.

You can confirm the font is installed with `fc-list` and grepping the list of fonts for the one you installed.

