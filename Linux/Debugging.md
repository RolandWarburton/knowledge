# Debugging Mega Section

### Add user details to github

This happens when github doesnt know who you are on a device.
Your email is the email that you registered to gihub with.
`git config --global user.email warburtonroland@gmail.com`.
The name is your github username
`git config --global user.name RolandWarburton`.

### Difference between & and &&

when you string commands together you have 2 options

* `command1 & command2`. command 1 is run in the background AND command2 runs at the same time
* `command1 && command2`. command1 has to be returned 0 (no errors) BEFORE command2 starts
* `command1; command2`. command1 will run THEN command2 will run after (sequential, waits for 1 to finish first)

### Using grep in a shell script runs for each word individually

```bash
# The problem...
mystring = "hello world"
grep "bleh" ${mystring}
```

outputs:

```none
"hello" => no matches found
"world" => no matches found
```

This is solved by piping the string into grep like you would do on the command line. [Source](https://unix.stackexchange.com/questions/163810/grep-on-a-variable)

```bash
# The solution
mystring = "hello world"
echo ${mystring} | grep "hello"
```

### Cannot write to files on NFS when its mounted through Thunar

This happens when an ftp connection is being used. this is unsecure so you cannot write.
However if you know you are using a secure connection it may be because of file permissions. The following steps are debug instructions to debug that.

1. Create a folder to permit access to `mkdir -p /home/myNFSFolder`
2. assign your *sftp group* access to the folder `sudo chgrp someGroup /home/myNFSFolder`
3. Make the fodler writable `sudo chmod 775 /home/myNFSFolder`
4. Set the GID for all subfolders too `sudo chmod g+s /home/myNFSFolder`
Make sure the user is in the correct group with the `groups 'username'`` command and add them if not.

### Thunar Freezes when a NFS drive is connected

Make sure `nfs-utils` is installed on the system. Once installed restart Thunar's daemon with `thunar -q` or kill it with `sudo pkill thunar`.
You can also try to unmount the drive with `sudo umount -af -t nfs`

### Wireless network not working (driver issues)

These steps are relevent when you run `lshw -c network` and your wireless card tells your either *\*-network UNCLAIMED* or *\*-network DISABLED*

1. Install/re-install `linux-firmware`.
2. Observe the output of `lshw -c network` and it should display `*-network DISABLED`. If not your driver is likely not supported as it was not contained in linux firmwares packages.
3. Enable the network with `sudo ip link set dev wlp3s0 up`. wlp3s0 is the name of my network card. You can also turn it on through some graphical interfaces such as [cmst](https://www.archlinux.org/packages/community/x86_64/cmst/)] for [connman](https://wiki.archlinux.org/index.php/ConnMan).

### StartX Debugging

#### xf86OpenConsole: Cannot open virtual console 1 (Permission denied)

This is caused by switching user. in particular when you log in as root then switch user.
To fix log out (or restart) then log back in as the user you want to run the startx command on.

#### startx failed to set iopl for i/o operation not permitted

Are you missing drivers?
I reinstalled `xf86-video-vesa` and installed `xf86-video-intel` (for my intel intergrated graphics laptop) and this solved startx not running.

Also make sure that you have a .xinitrc in your home directory. there is an example one in `/etc/X11/xinit/xinitrc`

### Remove old boot entries (created by boot managers)

You can do this installed in the live media off a usb or in the full environment if you need to.
Run `efibootmgr` to see the list of boot entries. and then `efibootmgr -b #### -B` -b to specify the boot id and -B to specify delete.

### Cannot remove EFI boot entries (re-appearing entries)

If you remove a boot entry as instructed above (Remove old boot entries) and reboot and it comes back.
This may be due to a BIOS entry called *"reserve memory for uefi boot manager"*.

As i understand it this option allows the boot entries to be stored in the firmware (NVRAM) rather then on the disk.
This means that when UEFI starts, the boot process may be looking at the firmware first and see the list of addresses and load them into the EFI directory (usually in /boot). For example the entry for grub may load itself to `EFI/GRUB/grubx64.efi`. Though when the *'reserve memory'* option is enabled and a boot entry is removed you will not be able to observe its '.efi' file being loaded. but will be able to observe its variable being loaded when running `efibootmgr` to list the boot entries that EFI knows about.

A Side effect/extra note is that your boot options list (pressing f12 to boot from a specific drive) will also not detect these hidden entries.

### How to image an iso to a usb (create bootable media)

Use dd (data duplicator).

* if = input
* of = output

`sudo dd status=progress if=/name/of/iso of=/dev/sdX`

Other common flags are.

* BS=(number)

### How to wipe a drive

* You can use the `wipe -r /dev/sdX`. command from the [repos](https://www.archlinux.org/packages/extra/x86_64/wipe/). This will do a quick erase of the disk.
* To entirely nuke a drive (similar to DBAN) use the `shred` command - `shred -v /dev/sdX`. -v for verbose. This is my preffered method as it avoids the "stack smashing" error that wipe sometimes throws (stack smashing is caused by buffer overflows, not sure why this happens though).

A common command for shredding a drive is to **-f**orce and only iterate once (-n1)

```none
sudo shred -f -n1 /dev/sdx
```

### Cant use echo in bash script

Use printf instead.

```bash
myvar="hello"
printf 'test '${myvar}' test';
```

### sudo open a file in Ranger

press *space* to select the file then press *@* and type *sudo nano*

### Updating NPM and NODE to latest versions

NPM already should be the latest version. verify the version at the [website](https://www.npmjs.com/package/npm)

NODE however is not the latest version. update it by installing Node Version Manager (NVM) ([install instructions](https://github.com/nvm-sh/nvm/blob/master/README.md)) and getting the latest version from the nvm script.

Once installed list avaliable versions with `nvm ls-remote` and install a version with `nvm install x.x.x`.

### applications showing unrecognized characters (placeholders)

Install the entire noto fonts package as a fallback for fonts. In particular `noto-fonts-cjk` for Chinese, Japanese, and Korean support.

```none
noto-fonts-emoji noto-fonts-cjk noto-fonts noto-fonts-extra noto-fonts
```

Then run `fc-cache -vf` to refresh your fonts and then reboot.

### VSCode cant save stuff as sudo

The regular official *code* package cannot do this according to the [wiki](https://wiki.archlinux.org/index.php/Visual_Studio_Code).
You can solve this by installing the AUR binary version thats contains proprietary stuff from microsoft [here](https://aur.archlinux.org/packages/visual-studio-code-bin/).

### tree command using weird Characters

tree looks like this

```none
.
|-- zprofile
`-- zshenv
```

But should look like:

```none
.
├── zprofile
└── zshenv
```

This is caused by a localisation issue. export these variables into your shell (.xsession or bashrc/zshrc).

```bash
export LANG=en_US.UTF-8
export LC_CTYPE=en_US.UTF-8
export LC_ALL=en_US.UTF-8
```

### Flameshot doesnt run

This issue happened when i switched over from a display manager (sddm) to xorg-xinit (~/.xinitrc).
`flameshot` froze the terminal and `flameshot gui` simply didnt start the application.

I think i solved this issue by changing my .xinitrc so that it uses dbus_launch to launch the desktop. `exec dbus-launch openbox-session`

### See list of uninstalled packages in pacman

```bash
grep 'removed' /var/log/pacman.log
```

### Boot stuck on *Reached target graphical interface*

This is an ongoing issue that i have had with my home computer. I havent solved it (it solved itself) yet but here is what i have done so far.

* Make journalctl log bigger. this avoids journalctl being *rotated* which can display when looking at the systemctl status of some process, ie. when it freezes and you ctrl+alt+f2 to look at the status of sddm to see what its doing. `sudo journalctl --verify` and `journalctl --vacuum-size=200M` to Delete old logs for debugging.
* Install the *nvidia* package.

### Download stuff off a website where you cant FTP in

Use a * in the filepath to **r**ecursively download all files from that point onwards.

`wget -r website.com/directory/*`

### Cannot find terminfo entry for 'some-terminal' when using screen over ssh

My solution to this was to install the same terminal on the host im connecting to.
This installs a terminfo file which you then need to export as a chosen terminal (export TERM=someterm)

### see the version of the OS i am on

`lsb-release -a` OR `hostnamectl`\
Rolling release distros like arch just say the release is 'rolling'. Whereas ubuntu has versions 'bionic', 'disco dingo', etc.

### see the kernal i am using

`uname`\
-r = release. (prints '5.4.1-arch1-1')\
-n = nodename/hostname (prints ...@'arch' or ...@'roland')\
-s = kernal (prints 'Linux')

### Figlet font locations

`/usr/share/figlet/fonts` (only .flf fonts seem to work)

### Recursive search folders and files for text

`grep -rw . -e 'TextToSearch'`\
-r = recursive\
-w = match the whole word\
-e = use this pattern
-l can be added to just give the file name of matching files

You can also use a file of patterns to check (one per line)\
`grep -rw . -f patterns.txt`

### Can't connect to ftp server (Connection refused)

I had this issue with a VPS once. The solution was to open up the ports for ftp again on the host.

1. Make sure you have vsftpd installed (vsftpd (Very Secure File Transfer Protocol Deamon) is an ftp daemon that creates a server to connect to over ftp.
2. Connect to the host through the web terminal provided by the VPS provider.

```none
firewall-cmd --permanent --add-port=21/tcp
firewall-cmd --permanent --add-service=ftp
firewall-cmd --reload
```

Furthermore there are a couple ways to check for open ports.
`sudo lsof -i -P -n | grep LISTEN`\
`sudo firewall-cmd --list-ports` and `sudo firewall-cmd --list-services`

### Errors When connecting to a ssh host and you get these errors when trying to use nano (eg. nano testfile > error)

`ERROR: /bin/sh: 1: /usr/bin/sensible-editor: not found`
`ERROR: Error opening terminal: xterm-kitty.`

```bash
# Solution
export TERM=xterm
```

### Backspace and Tab are spaces instead when in SSH

Reset the terminal variable

```bash
# ~/.bashrc
export TERM=xterm
```

### Check what filesystem type you are using

`lsblk -f` -f outputs info about filesystems

### My Time and Date are all fucked up

1. Verify your timezone `timedatectl status`.
2. See all available timezones with `timedatectl list-timezones`
3. Finally, set your timezone with `sudo timedatectl set-timezone Australia/Melbourne`.

If that doesnt work you should run `timedatectl set-ntp true` to set your network time back to true so it gets the time correctly.

```none
# WRONG
Time zone: UTC (UTC, +0000)
# CORRECT
Time zone: Australia/Melbourne (AEDT, +1100)
```

### What does curl piped into -E Mean

```none
curl -sL https://deb.nodesource.com/setup_13.x | sudo -E bash -
```

`Curl -s (silent) -L` (follow redirects). Then pipe it and execute the script from the url in into bash and preserve the existing environment variables (-E), Finally the - means "the thing thats being piped".

##### Further explanation of the pipe part of the command

`someCommand | **sudo** -E **bash** -`\
Pipe the command into bash (like running it as sudo in bash normally).\
`someCommand | sudo **-E** bash -`\
Preserve the environment variables in my bash environment\
`someCommand | sudo -E bash **-**`\
Substitute the result of someCommand into '-'

### NVENC codec - OBS

The error `Failed to open NVENC codec: Unknown error occurred` occurs when you have updated your linux version and not restarted.

To verify that this is causing the issue run the commands `pacman -Q linux` and `uname -r` and check that they match.

If they do not match, then to resolve this you can restart and in 9/10 cases it will fix and they will match.
