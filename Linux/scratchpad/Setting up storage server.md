# Setting up storage server

After installing debian 10 Buster. I went ahead with these sections.

## Networking

Using NetworkManager nmtui i created these settings to access my router over the vlan 10 network (10.10.10.0/24).
the PC doesn't require any knowledge of what vlan its on.

![nmtui ui dialogue](https://i.imgur.com/uvLc5uv.png)

Then disable the debian interfaces config to reply only on network manager.

```none
sudo mv /etc/network/interfaces /etc/network/interfaces.save
```

Then ensure that network manager is started correctly etc.

```none
sudo systemctl start NetworkManager
sudo systemctl status NetworkManager
sudo systemctl enable NetworkManager
```

Then set up my layer 2 switch to move this traffic, i'm not sure what this is doing (brute force solution).

[edge switch vlan config](https://i.imgur.com/Oa0MvP3.png)

* I am setting the port of my new server (port 3) to "Untagged" - ie. Native Vlan of this port is now vlan 10.
* I set the switches default vlan 1 (port 3) to "Exclude" - ie. do not participate in this, The port is never a member of the selected VLAN (registration forbidden).
* I leave the rest of vlan 10's ports as "Tagged" - ie. members of vlan 10 trunk.

## Basic setup

udisks provides a daemon udisksd, that implements D-Bus interfaces used to query and manipulate storage devices, and a command-line tool udisksctl, used to query and use the daemon.

This should suppress the warning when looking under the storage tab on cockpit.

```none
sudo systemctl start udisks2
```

## Monitoring

### Cockpit

Next i want to easily monitor different services and the status of the machine so in install *cockpit*

```none
sudo apt install cockpit
sudo systemctl enable cockpit
```

server_ip:9090 will now serve cockpit.

### sysstat

using `iostat` command from sysstat i am monitoring the disk activity as i was noticing high usage at idle on my mounted NAS drives (6 mib/s).

First install sysstat.

```none
sudo apt install sysstat
```

Then enable it

```none
/etc/default/sysstat
---
# Should sadc collect system activity informations? Valid values
# are "true" and "false". Please do not put other values, they
# will be overwritten by debconf!
ENABLED="true"
```

Then restart sysstat.

```none
sudo service sysstat restart
```

Then observe any problems using `iostat`. In my situation i can see that the write speed is unusually high (~6 kB_wrtn/s at idle) however i dont have an explanation for this right now.

Some other iostat commands...

* `iostat -p sda` - Target one device only (-p device)
* `iostat -N` - Summarize LVM (no partitions, sda, sdb only)

## File shares

I plan on using samba for shares.

---

[https://wiki.debian.org/NetworkConfiguration#Setting_up_an_Ethernet_Interface](https://wiki.debian.org/NetworkConfiguration#Setting_up_an_Ethernet_Interface)

/etc/network/interfaces

<!-- touch eno1.10.netdev &&
touch eno1.10.network &&

eno1.10.netdev

```none
[NetDev]
Name=eno1.10
Kind=vlan

[VLAN]
Id=10
```

eno1.10.network

```none
[Match]
Name=eno1.10

[Network]
DHCP=no

[Address]
Address=10.10.10.15/24
``` -->

## Misc drive management tools

Other than the `iostat` and other sysstat commands like...

sysstat normally takes a sample every X seconds, Y times. This is what the 2 numbers mean on the each of sysstat commands.
If there is just 1 number it means the command will take N samples every 1 seconds forever.

### pidstat - Process ID statistic monitoring

* `pidstat` - See running processes (CPU)
* `pidstat -d 2` - See running IO related process (2 is the refresh interval)
* `pidstat -R` - See realtime priority and scheduling information

### sar - System activity statistic monitoring

Use `sar` to schedule cron reporting. For these examples we will run manually.

* `sar -u -o sarfile 2 5` - Read cpu usage (-u) and save it to ./sarfile (bin file) in the current directory
* `sar -dh -o sarfile 2 5` - Read disk usage (-d disk usage) (-h human readable (see disk monitoring in more details))
* `sar -F 2 4` - Report disk usage (how full each partition (sda1, sdb2, etc) is)
* `sar -n DEV 1 3` - Report network usage (DEV is not a placeholder, use it literally in the command)
* `sar -rh 1 3` - Report RAM usage

### sar disk monitoring in more details

A command like `sar -d --human 1 3` is good at showing you numbers, but what if you need to map a volume like `dev8-0` or `dev8-16` back to its `/dev/sdx` name.

This can be done (technically speaking) through `ls -l /dev/sd*` and looking at the major (8) numbers and the minor numbers (16, 32, 48, etc).

```output
roland@debian:~$ ls -l /dev/sd*
brw-rw---- 1 root disk 8,  0 Dec 28 14:14 /dev/sda
brw-rw---- 1 root disk 8,  1 Dec 28 14:14 /dev/sda1
brw-rw---- 1 root disk 8,  2 Dec 28 14:14 /dev/sda2
brw-rw---- 1 root disk 8,  3 Dec 28 14:14 /dev/sda3
brw-rw---- 1 root disk 8,  4 Dec 28 14:14 /dev/sda4
brw-rw---- 1 root disk 8, 16 Dec 28 14:14 /dev/sdb
brw-rw---- 1 root disk 8, 17 Dec 28 14:14 /dev/sdb1
brw-rw---- 1 root disk 8, 32 Dec 28 14:14 /dev/sdc
brw-rw---- 1 root disk 8, 33 Dec 28 14:14 /dev/sdc1
brw-rw---- 1 root disk 8, 34 Dec 28 18:07 /dev/sdc2
brw-rw---- 1 root disk 8, 48 Dec 28 14:14 /dev/sdd
```

However, there is a better way! Simply use the `-p` flag with sar and it will do the translation for you.

```none
sar -dp --human 1 3
```

```output
Average:          DEV       tps     rkB/s     wkB/s   areq-sz    aqu-sz     await     svctm     %util
Average:          sda      0.00      0.0k      0.0k      0.0k      0.00      0.00      0.00      0.0%
Average:          sdb    116.00      0.0k    110.1M    972.1k      3.65     32.23      5.05     58.5%
Average:          sdc    954.33    118.1M      0.0k    126.7k      1.53      1.62      1.03     98.1%
Average:          sdd      0.00      0.0k      0.0k      0.0k      0.00      0.00      0.00      0.0%
```

Read a sarfile with `sar -f ./sarfile`.

## Formatting drives

### Format >2TB drives to GPT (set drive label to GPT)

```none
parted /dev/sdb
```

```none
print
```

```output
Model: ATA ST4000VN008-2DR1 (scsi)
Disk /dev/sdb: 4001GB
Sector size (logical/physical): 512B/4096B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  2199GB  2199GB  primary
```

```none
mklabel gpt
```

```output
Warning: The existing disk label on /dev/sdb will be destroyed and all data on this disk will be lost. Do you want to continue?
Yes/No? Yes
```

```none
print
```

```output
Model: ATA ST4000VN008-2DR1 (scsi)
Disk /dev/sdb: 4001GB
Sector size (logical/physical): 512B/4096B
Partition Table: gpt
Disk Flags:

Number  Start  End  Size  File system  Name  Flags
```

### Create and write the partition

```none
mkpart primary 0GB 100%
```

```none
print
```

```output
Model: ATA ST4000VN008-2DR1 (scsi)
Disk /dev/sdb: 4001GB
Sector size (logical/physical): 512B/4096B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name     Flags
 1      1049kB  4001GB  4001GB               primary
```

now that you have written your new changes, you will almost certainly need to update fstab with a new UUID (it changes after writing to a fresh drive). I will do that step later.

### Observe the changes in fdisk

See that its now `Disklabel type: gpt`, `Type Linux filesystem`, and its size is >2TB.

```none
sudo fdisk /dev/sdb
print
```

```output
Disk /dev/sdb: 3.7 TiB, 4000787030016 bytes, 7814037168 sectors
Disk model: ST4000VN008-2DR1
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: 917906C6-C578-48C6-BB61-00168987285D

Device     Start        End    Sectors  Size Type
/dev/sdb1   2048 7814035455 7814033408  3.7T Linux filesystem
```

### Format the drive as ext4

```none
sudo mkfs.ext4 /dev/sdb1
```

```output
mke2fs 1.44.5 (15-Dec-2018)
Creating filesystem with 976754176 4k blocks and 244195328 inodes
Filesystem UUID: 06379adf-0507-4cc0-b33f-910516a9f245
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968,
        102400000, 214990848, 512000000, 550731776, 644972544

Allocating group tables: done
Writing inode tables: done
Creating journal (262144 blocks): done
Writing superblocks and filesystem accounting information:
done
```

### Create the drive mount point

The drive can be mounted anywhere, but traditionally it could go in `/mnt`.

I verify the drive ID and create a folder in `/mnt` to easily identify the drive later in fstab.

sudo mkdir /mnt/06379adf-0507-4cc0-b33f-910516a9f245

### Add entry to fstab

```none
sudo vim /etc/fstab
```

Add the following line, change the drive id to your UUID.

```none
UUID=06379adf-0507-4cc0-b33f-910516a9f245       /mnt/06379adf-0507-4cc0-b33f-910516a9f245       auto nosuid,nodev,nofail,x-gvfs-show 0 0
```

* auto - automatically mounts the partition at boot
* nosuid - specifies that the filesystem cannot contain set userid files. This prevents root escalation and other security issues.
* nodev - specifies that the filesystem cannot contain special devices (to prevent access to random device hardware).
* nofail - removes the errorcheck. (in case the drive is missing)
* x-gvfs-show - show the mount option in the file manager. If this is on a GUI-less server, this option won't be necessary.
* 0 - determines which filesystems need to be dumped (0 is the default).
* 0 - determine the order in which filesystem checks are done at boot time (0 is the default).

### Dry run mount

Now that the system is almost ready to reboot, perform a dry run by mounting all the entries in `/etc/fstab` using the `sudo mount --all` command.

If you get no errors, have a look in your mount point. Also observe your new space with `df -h`.

```output
Filesystem      Size  Used Avail Use% Mounted on
...omitted output...
/dev/sdb1       3.6T   89M  3.4T   1% /mnt/06379adf-0507-4cc0-b33f-910516a9f245
```

With all those checks passing, its safe to reboot and ensure the drive auto mounts on boot.
