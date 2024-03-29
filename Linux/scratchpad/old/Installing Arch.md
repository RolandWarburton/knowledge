# Getting Arch going

This section is a loose collection of the steps i take when setting up my system from scratch. I expect these steps to become out of date very quick as i learn new faster/better ways of doing things as i continue learning.

### Installing Arch

Not going to explain entirely here. The wiki is your best friend :)

Checklist of packages that i like to install during base package installation:

* [x] base, base-devel, linux - All required by default
* [x] nano - Always good to have a backup to vim
* [x] dhcpcd - Arch doesnt come with any networking tools
* [x] openssh - So i can ssh in after install

Creating the roland user:

```none
useradd -g users -G wheel,storage,power -m roland
```

* Rolands primary group is users
* His secondary groups are wheel(sudo),storage, and power
* Create a home directory folder for Roland with `-m`
* Dont forget to allow roland (and other members of wheel) access to sudo by enabling access in `/etc/sudoers`

#### Getting the desktop going

After installing your WM/DE of choice copy startx config from `/etc/X11/xinit/xinitrc` to `~/.xinitrc` and then refer to [startx debugging](https://github.com/RolandWarburton/knowledge/Debugging) because i dont think i have ever gotten startx to work first try without wanting to poke my eyes out.

#### Setting a hostname

Edit /etc/hostname and give your computer a name. This will be in the format of `user@hostname`.

Once you have changed your hostname file. add that host to `/etc/hosts` and add a host entry for a connection when you ssh into the machine. For example:

```none
127.0.0.1 localhost
127.0.1.1 MyCustomHostname # <- change to the same name that you set in /etc/hostname
```
