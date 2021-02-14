# Samba shares

## Diagnostics

Check config file syntax.

```none
testparm /etc/samba/smb.conf
```

List all users

```none
sudo smbstatus
```

list a specific users details as the logged in user

```none
sudo pdbedit -L -v
```

* -L, --list                            list all users
* -v, --verbose                         be verbose

## Administration

Add users

```none
smbpasswd -a roland
```

Check the users and groups on the linux system locally.

```none
# see all users
less /etc/passwd

# see currently logged in user
whoami

# see all groups
less /etc/group

# see currently logged in users groups
groups
id

# see who's logged in
w
```

## Standalone authentication example

Resources and tutorials:

* From the [docs](https://wiki.samba.org/index.php/Setting_up_Samba_as_a_Standalone_Server#Creating_a_Basic_authenticated_access_smb.conf_File).
* [Setting up anon shares on CentOS 8](https://www.linuxtechi.com/install-configure-samba-centos-8/)

By default, Samba uses the tdbsam to store users in a database located at `/urs/local/samba/private/passdb.tdb`.

### Create a local user account

First you need to have a local user to add to the samba database. Create a local users with the useradd tool.

```none
sudo useradd --no-create-home --shell /sbin/nologin demoUser

# give the demoUser a password
# by default the users will be in the group demoUser as well automatically
sudo passwd demoUser
```

### Add the local users to the samba database

```none
# add the user
sudo smbpasswd -a demoUser

# enable the user
smbpasswd -e demoUser
```

### Create POSIX ACLs to configure read/write access

Set the following POSIX permissions on the desired share folders

```none
sudo chmod -R 0755 /srv/samba/shared
sudo chown -R nobody:nobody /srv/samba/shared

# if running selinux set this
sudo chcon -t samba_share_t /srv/samba/shared
```

## Sample configs

### CentOS 8 anonymous and home shares

The global config was taken from [here](https://askubuntu.com/questions/258284/setting-up-an-anonymous-public-samba-share-to-be-accessed-via-windows-7-and-xbmc) and is a good verbose starting point.

The home directory is hidden using `browseable = No` and can be navigated to using `smb://server/username`, for my own reference/example `smb://sftp/roland`

```none
[global]
server string = YOURSERVERNAME
workgroup = WORKGROUP
netbios name = samba
guest account = root
smb ports = 445
#max protocol = SMB2
min receivefile size = 16384
deadtime = 30
os level = 20
mangled names = no
name resolve order = lmhosts wins bcast host
preferred master = auto
domain master = auto
local master = yes
printcap name = /dev/null
load printers = no
browseable = yes
writeable = yes
printable = no
enable core files = no
passdb backend = smbpasswd
smb encrypt = disabled
use sendfile = yes

[anonymous]
comment = Share
path = /srv/samba/anonymous
available = yes
browsable = yes
writable = yes
public = yes

[homes]
comment = Home Directories
path = /home/%S
valid users = %S
read only = No
create mask = 0700
directory mask = 0700
browseable = No
```

### Debian 10 shared homes, anonymous shares, and protected shares

The global config i stuck together myself (with duck tape and the tutorials listed above).
You need to do extra config steps such as adding the user first for this to work.

```none
[global]
server string = samba
workgroup = WORKGROUP
netbios name = samba
wins support = yes

# types of users permitted (never | bad user)
map to guest = bad user

# security
security = user
hosts allow = 192.168.0.0/24 10.10.10.0/24 localhost
hosts deny = 0.0.0.0/0

# logging
log file = /var/log/samba/%m
log level = 1

[homes]
comment = Home Directories
valid users = %S
read only = no
create mask = 0755
directory mask = 0755
browseable = yes
veto files = /*.*/

[anonymous]
comment = anonymous
path = /srv/samba/anonymous
read only = no

create mask = 0644
force create mode = 0644
directory mask = 0755
force directory mode = 0755
force user = roland
force group = roland

public = yes
writable = yes

[protected]
comment = protected
path = /srv/samba/protected
read only = no
```

<!-- sudo chgrp -R demoGroup /srv/samba/guest
sudo chgrp -R demoGroup /srv/samba/demo

sudo chmod 2775 /srv/samba/guest
sudo chmod 2770 /srv/samba/demo -->

### Sharing homes (improved version)

This version is different one from the default provided by samba (in debian 10).
My fixed version changes the write permissions to fix the "unable to write files/dirs" issue.

```none
[homes]
comment = Home Directories
path = /home/%S
valid users = %S
read only = No
create mask = 0700
directory mask = 0700
browseable = No
# follow symlinks
follow symlinks = yes
wide links = yes
```

### Fix error opening files

Sometimes depending on the computer, you will be able to browse to a share, but not open it in any external editor (for example feh).

One possible fix for this is to install the gvfs packages to try and make it work.

* gvfs
* gvfs-fuse
* gvfs-bin
* gvfs-daemons
* gvfs-libs
* gvfs-libs

Then make sure to **restart** after installing these to give a chance for gvfs to switch on.

The next thing you can do is to make sure you are on the "stable" branch of your distribution. I have had this issue (probably) caused by packages mismatching, or some other package version related issues. Make sure to remove any mention of "testing" from your `/etc/apt/sources.list`. In otherwords change this `deb http://deb.debian.org/debian/ testing main contrib non-free` to this `deb http://deb.debian.org/debian/ main contrib non-free`.

Another thing to check is your kernel.hostname.

```none
sudo sysctl kernel.hostname
```

You can change it like this.

```none
sudo sysctl kernel.hostname=samba
```

Note that i strongly believe the first, and second solution here solves the problem, but i included this one as well just to make sure.

