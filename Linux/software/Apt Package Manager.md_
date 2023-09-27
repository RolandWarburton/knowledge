# Debian and apt based systems

### downloading a package from a source (16.04 xenial only)

browse the website [launchpad](https://launchpad.net). For example [https://launchpad.net/ubuntu/+source/kitty](https://launchpad.net/ubuntu/+source/kitty).\
Once you find a package go to your debian system and type `sudo apt-add-repository ppa:kitty/ppa`.\
Then sync your packages `sudo apt-get update`. Then install it with `sudo apt-get install kitty`.

In the event that you get the error: *The repository '[http://ppa.launchpad.net/kitty/ppa/ubuntu](http://ppa.launchpad.net/kitty/ppa/ubuntu) bionic Release' does not have a Release file.*\
This means the package is not built for that version of debian/ubuntu. Look for alternate versions under *Other versions of 'somepackage' in untrusted archives.*.

To remove the ppa use `sudo add-apt-repository --remove ppa:mc3man/trusty-media` then `sudo apt-get update`.

### Remove a package entirely

Purge `sudo apt-get purge packageName*`\
Remove logs (if any) `sudo rm -r /var/log/packageName`\
Remove other files (if any) `sudo rm -r /var/lib/packageName`

### Upgrade and update packages

Update = get new versions of packages. Dont install them.\
Upgrade = Install any new updated packages.\
run update first then upgrade.\
`sudo apt update` then `sudo apt upgrade`.

### Upgrade the entire distribution

Install the new release for the upgrade `sudo apt-get dist-upgrade`\
Run the update `sudo do-release-upgrade`

### Maintaining APT packages

#### HIT, IGN, and GET meaning

When you run `sudo apt update`. What do these things mean?
**Hit** = Package didnt change since the last check. there is no newer version of the package.\
**Get** = This means there is a package update(new version) available and it will download the details for this update, but not the update itself.\
**Ign** = This means that the package has been ignored. This happens either because of an error or because the package is recent and there is no need to check it for updates.

### Third party stuff

Every third party package has a key (called an APT or GPT key). review the list of keys with `sudo apt-key list`\
The last 8 digits is the key ID (058F 8B6B)

```none
/etc/apt/trusted.gpg
--------------------
pub   rsa4096 2018-04-18 [SC] [expires: 2023-04-17]
      E162 F504 A20C DF15 827F  718D 4B7C 549A 058F 8B6B
uid           [ unknown] MongoDB 4.2 Release Signing Key <packaging@mongodb.com>
```

Remove this key with `sudo apt-key del 058F8B6B`

All of your third party sources should be in `/etc/apt/sources.list.d/*` with seperate files for each source.\
All of your ubuntu packages and packages from the ubuntu repos should go in `/etc/apt/sources.list`

### Process of installing a third party package

1. import the key with `sudo apt-key add` from some script.\
Example: `wget -qO - https://www.mongodb.org/static/pgp/server-4.2.asc | sudo apt-key add -`
2. Create a source file in `/etc/apt/sources.list.d/`\
Example: `echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.2.list`. The use of the 'tee' command means that the command is redirected into stdout and a file. Ie. echo it and put it in the file.

### see big list of all sources installed

not sure what it is tho or what it means, or what its important for.

`ls /var/lib/apt/lists`
If you delete this folder and run `apt get update` again it redownloads all of the files again.

## Snap Packages

Snap is an alternative way to package software. It was developed by canonical (who are the creators of ubuntu) and as such is on ubuntu/ubuntu server and ubuntu derivitives, snap can also be installed on other non ubuntu related distributions such as debian or arch (aur) where it is not installed by default by installing the *snapd* package.

You can check what snap packages are currently installed with `snap list`.
Example Output:

```none
Name  Version    Rev   Tracking  Publisher   Notes
code  26076a4d   23    stable    vscode✓     classic
core  16-2.42.5  8268  stable    canonical✓  core
```

You can search for information about a package, such as its channels (releases, eg stable/latest/beta etc)

### Managing/Updating snap packages

You can check the last time a refresh (update) was run with `snap refresh --time`. and look at what changes were made with `snap changes`

By defaut Snap packages check for updates 4 times a day and will update themselves. You can change this by running
`sudo snap set system refresh.timer=4:00-7:00,19:00-22:10` and defining your own times.

Manually updating a snap package: `snap refresh myPackage` or check for updates on all packages with `snap refresh`.

### Installing nvidia drivers

Refer to the [Nvidia Graphics Drivers](https://wiki.debian.org/NvidiaGraphicsDrivers) documentation here.

#### Update kernel

Because i am on debian buster testing branch i need to update my kernel from debian backports using the `-t` flag.

```none
apt install -t buster-backports linux-headers-amd64
```

This configuration for `/etc/apt/sources.list` seems to have the correct sources to install the nvidia-driver into.

The guide from debian seems to want you to use `deb http://deb.debian.org/debian buster-backports main contrib non-free` especially (for buster backports).

```none
deb http://deb.debian.org/debian/ testing main non-free
deb-src http://deb.debian.org/debian/ testing main non-free

deb http://security.debian.org/debian-security buster/updates main
deb-src http://security.debian.org/debian-security buster/updates main

# buster-updates, previously known as 'volatile'
deb http://deb.debian.org/debian/ buster-updates main non-free
deb-src http://deb.debian.org/debian/ buster-updates main non-free

# This system was installed using small removable media
# (e.g. netinst, live or single CD). The matching "deb cdrom"
# entries were disabled at the end of the installation process.
# For information about how to configure apt package sources,
# see the sources.list(5) manual.

deb http://deb.debian.org/debian buster-backports main contrib non-free
deb-src http://ftp.au.debian.org/debian/ buster main non-free
```

Then run this command to target the buster-backports repo and install the required stuff.

```none
sudo apt update
sudo apt upgrade
apt install -t buster-backports nvidia-driver firmware-misc-nonfree
```
