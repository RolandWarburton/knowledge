# LVM Notes

Traditionally without LVM you have three layers of storage:

1. Hard drive: `/dev/sda`
2. Partitions: `/dev/sda1`
3. File system: `/dev/sda1` formatted as EXT4 mounted at `/`

LVM adds three layers between the **Partitions** and the **File system**.

1. PV: Physical Volume - This partition `/dev/sda1` will become a LVM physical volume that needs to be mapped to a volume group
2. VG: Volume Group - This partition `/dev/sda1` will be mapped to the volume group `datavg0`
3. LV: Logical Volume - This volume group will have one partition EXT4 `/dev/datavg0/lv0`

![lvm diagram](https://d33wubrfki0l68.cloudfront.net/cda83679d927d9fbbdd73e2656b5615e2e2e6c89/81d1e/images/2019/11/lvm-key.jpeg)

## Installing

Install the `lvm2` package on Debian.

```none
sudo apt install lvm2
```

## Getting the lay of the land

Use the `lvmdiskscan` to get a list of all the volumes and partitions.

I have quite a few physical drives but as far as lvm is concerned i only have 1 disk.

```output
roland@store:~$ sudo lvmdiskscan
  /dev/sda1 [     512.00 MiB]
  /dev/sda2 [    <457.58 GiB]
  /dev/sda3 [       7.68 GiB]
  /dev/sdb1 [      <3.64 TiB]
  /dev/sdc1 [     931.51 GiB]
  /dev/sdd  [      <7.28 TiB]
  /dev/sde1 [      <3.64 TiB]
  /dev/sdf1 [     200.00 MiB]
  /dev/sdf2 [      <2.73 TiB]
  /dev/sdg1 [      <7.62 GiB]
  1 disk
  9 partitions
  0 LVM physical volume whole disks
  0 LVM physical volumes
```

## Setting up a single drive

### Formatting large drives to EXT4

I will only be setting up a volume group of one physical drive that will be fully dedicated to the volume group.

The drive that will be configured is `/dev/sdd`. I will format the drive as `ext4` that will be 100% of the device.

For this task (formatting) i will need to use `parted` because the size of the drive is >2T.

```none
sudo parted /dev/sdd
```

From within parted enter these commands. This will wipe any existing partitions and create a single partition that will be 100% of the drive.

```none
mklabel gpt

mkpart primary 0GB 100%
```

Then press `ctrl+d` to exit parted.

the command `lsblk` should now read out the new partition.

```output
lsblk
...
sdd      8:48   0   7.3T  0 disk
└─sdd1   8:49   0   7.3T  0 part
...
```

Next format the drive with `mkfs.ext4`. You **must** use EXT4 for your drives that use LVM for compatability reasons with LVM.

```none
sudo mkfs.ext4 /dev/sdd1
```

verify the changes with `fdisk` and `lsblk`.

```none
sudo fdisk /dev/sdd
print
```

* Note that: `Disklabel type: gpt` should be gpt.
* Note that the disk ID is `F404B3AB-2161-4412-A651-75AD9E498BFE`

```output
Disk /dev/sdd: 7.28 TiB, 8001563222016 bytes, 15628053168 sectors
Disk model: ST8000VN004-2M21
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: F404B3AB-2161-4412-A651-75AD9E498BFE

Device     Start         End     Sectors  Size Type
/dev/sdd1   2048 15628052479 15628050432  7.3T Linux filesystem
```

* Note that the filesystem type is `ext4` with `lsblk -f`

```output
sdd
└─sdd1 ext4   1.0         ce179777-d4db-4497-b4d4-f2d186bd0ba0
```

Now we have a drive that is formatted as EXT4.

### Layer 1 - Setting up the physical volume group

```none
sudo pvcreate /dev/sdd1
```

### layer 2 - Setting up the volume group

First check for existing volume groups. The following command should not return anything if there are no volume groups created yet.

```none
sudo vgscan
```

Create a new logical volume group. If you have more than one drive then add them to the end of the command.

```none
sudo vgcreate datavg0 /dev/sdd1
```

We can observe the newly created physical volumes with `pvdisplay` and `pvs`.

* We can see that `/dev/sdd1` is a member of the volume group `datavg0`

```output
# pvdisplay
  --- Physical volume ---
  PV Name               /dev/sdd1
  VG Name               datavg0
  PV Size               <7.28 TiB / not usable 4.00 MiB
  Allocatable           yes
  PE Size               4.00 MiB
  Total PE              1907720
  Free PE               1907720
  Allocated PE          0
  PV UUID               jynvMg-xFeC-AiW1-E83C-A1vZ-Zvu1-nbwPhN

# pvs
  PV         VG      Fmt  Attr PSize  PFree
  /dev/sdd1  datavg0 lvm2 a--  <7.28t <7.28t
```

We can observe the newly created volume groups with `vgdisplay` `vgs`

* We can see that `datavg0` has one physical volume in it.
* We can see that datavg0 has no logical volumes created for it yet (Cur LV is 0).

```none
# vgdisplay
  --- Volume group ---
  VG Name               datavg0
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <7.28 TiB
  PE Size               4.00 MiB
  Total PE              1907720
  Alloc PE / Size       0 / 0
  Free  PE / Size       1907720 / <7.28 TiB
  VG UUID               NCaRH5-cNWf-2Ppu-eeFc-cDkP-deNQ-QNi5Hv

# vgs
  VG      #PV #LV #SN Attr   VSize  VFree
  datavg0   1   0   0 wz--n- <7.28t <7.28t
```

We can also now see that `/dev/sdd1` is a LVM physical volume using `lvmdiskscan` again.

```output
  /dev/sda1 [     512.00 MiB]
  /dev/sda2 [    <457.58 GiB]
  /dev/sda3 [       7.68 GiB]
  /dev/sdb1 [      <3.64 TiB]
  /dev/sdc1 [     931.51 GiB]
  /dev/sdd1 [      <7.28 TiB] LVM physical volume
  /dev/sde1 [      <3.64 TiB]
  /dev/sdf1 [     200.00 MiB]
  /dev/sdf2 [      <2.73 TiB]
  /dev/sdg1 [      <7.62 GiB]
  0 disks
  9 partitions
  0 LVM physical volume whole disks
  1 LVM physical volume
```

#### Adding a new drive to the volume group

If you have more drives to add the the volume group `datavg0` then use this command.

```none
sudo vgextend datavg0 /dev/sd[letter][number]
```

Then resize the logical volume.

```none
sudo lvm lvextend -l +100%FREE /dev/datavg0/root
```

Then resize the filesystem.

```none
sudo resize2fs -p /dev/mapper/datavg-root 
```

#### Removing drives from the volume group

To remove the drive from the volume group use the following commands.

```none
# remove the logical volume
lvremove /dev/datavg0/newvol

# run this on each physical volume until there is only one left
vgreduce datavg0 /dev/sdd1

# remove the last physical volume by just removing the whole volume group
sudo vgremove datavg0

# remove the volume group/s
sudo pvremove /dev/sdd1
```

### Layer 3 - Setting up the locical volumes

First some more terminology and a recap of terms.

* VG - Volume Group - The highest level of abstraction that encompasses a set physical volumes, and exposes logical volumes as partitions file systems.
* PV - Physical Volume - A disk partition that is assigned to a volume group.
* LV - Logical Volume - A logical partition on a volume group.

Extents are a way of splitting up data. [source from tldp](https://tldp.org/HOWTO/LVM-HOWTO/tyingittogether.html).

* PE - Physical Extent -  Each physical volume is divided chunks of data, known as physical extents, these extents have the same size as the logical extents for the volume group.
* LE - Logical Extent -  Each logical volume is split into chunks of data, known as logical extents. The extent size is the same for all logical volumes in the volume group.

Take this example to understand PE and LE. PV1 and PV2 are physical volumes that belong to the same volume group.

When the **logical volume** is created a mapping is defined between **logical extents** and **physical extents**, eg. logical extent 1 could map onto physical extent 51 of PV1, so when data is written to the first 4 MB of the **logical volume** it will in fact be written to the 51st extent of PV1.

Now that we understand that logical extents (LE) map to physical extends (PE), we can understand mapping modes.

* Linear mapping - PEs are mapped to an area of a LV in a linear way.
  * eg. LE 1 to 99 maps to PE 1 to 99
  * eg. LE 99 to 100 maps to PE 300 to 400
* Striped mapping - LEs are mapped to a set of PEs in a striped way.
  * LE 1 maps to PV1 1
  * LE 2 maps to PV2 1
  * LE 3 maps to PV1 2
  * LE 4 maps to PV2 2

There is a caveat to this. The number of mappings gets confusing when you exceed the number of PEs in the smallest PV. This can be overcome in lvm2 but i do not understand how. Read more about the caveat [here](https://tldp.org/HOWTO/LVM-HOWTO/mapmode.html) if you care about striping LVM.

Regardless of this, i will NOT be using striping because i don't care much for performance and hassle with the LVM mapping. I will be using linear mapping instead, especially since i only have one drive. Also importantly, raid is not a backup and LVM striping is not even raid so there is not much to gain in my situation.

#### Command Examples - Creating LV

* `-L` sets a fixed size and `-l` indicates a percentage of the remaining space in the VG.

To create a logical volume called newvol that takes up 90% of the volume group space.

```none
sudo lvcreate -l 90%VG --name root datavg0
```

To create a new logical volume called newvol that takes up 2GB of the volume group space.

```none
sudo lvcreate -L 2G --name newvol datavg0
```

To remove a lv called newvol in the datavg0 VG.

```none
lvremove /dev/datavg0/newvol
```

#### Extending the LV

Extend LV to 12G the very robust and old way of doing things.

```none
lvextend -L12G /dev/datavg0/newvol
```

The better way of doing things, extend the LV to 12G.

```none
lvresize --size -L12G /dev/datavg0/newvol
```

Remove 2G from the LV.

```none
lvresize --size -2G /dev/datavg0/newvol
```

If you have a filesystem use the `--resizefs` flag.

```none
mkfs.ext4 /dev/datavg0/newvol

# subtract 2G from the total LV size
lvresize --resizefs --size -2G /dev/datavg0/newvol

# add 2G to the total LV size
lvresize --resizefs --size -2G /dev/datavg0/newvol

# set the LV size to 12G
lvresize --resizefs --size 12G /dev/datavg0/newvol

# set the LV size to 100% of the VG
lvresize --resizefs -l 100%VG /dev/datavg0/newvol
```

#### Renaming the LV

To find the current name use `lvs`.

```output
LV   VG      Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
borg datavg0 -wi-ao---- <6.55t
```

I called my LV "borg" but want to rename it.

Use the `lvrename` command to rename the LV.

```none
lvrename datavg0 borg newname
```

## LVM Raid 1

LVM can be used for raid 1 (and probably others but i'm not interested in setting that up) to create some redundancy in my data.

I'm following a tutorial from gentoo wiki to create LVM and Raid 1. Link [here](https://wiki.gentoo.org/wiki/Raid1_with_LVM_from_scratch).

I will be operating on two identical 8TB drives. `/dev/sdc` and `/dev/sdd`.

### Prepping The Disks

#### Configuring SDC Partitions

Using parted to create partitions. Start parted with the -a optimal flag to use the "optimum alignment as given by the disk topology".

```none
parted -a optimal /dev/sdc
```

Set the label as GPT.

```none
mklabel gpt
```

Create a primary partition and use all the space on that disk (`-1`) on the primary partition number `1`.

```none
mkpart primary 1 -1
```

Set partition name to raiddata0 on partition 1.

```none
name 1 raiddata0
```

Enable the LVM flag on partiton 1.

```none
set 1 lvm on
```

Verify your settings with the `print` command from within parted.

```output
Model: ATA ST8000VN004-2M21 (scsi)
Disk /dev/sdc: 8002GB
Sector size (logical/physical): 512B/4096B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name       Flags
 1      1049kB  8002GB  8002GB               raiddata0  lvm
```

Press `ctrl+d` to exit parted.

#### Configuring SDE Partitons

Repeat above, replace `/dev/sdc` with `/dev/sde`, and `raiddata0` with `raiddata1`.

```none
roland@store:~$ sudo parted -a optimal /dev/sdd
GNU Parted 3.4
Using /dev/sdd
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) mklabel gpt
(parted) mkpart primary 1 -1
(parted) name 1 raiddata1
(parted) set 1 lvm on
(parted) print
Model: ATA ST8000VN004-2M21 (scsi)
Disk /dev/sdd: 8002GB
Sector size (logical/physical): 512B/4096B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name       Flags
 1      1049kB  8002GB  8002GB               raiddata1  lvm

(parted)

Information: You may need to update /etc/fstab.
```

### Setting up LVM

#### Create And Raid Volume Groups

Create the LVM **physical** volumes for both physical disks

```none
lvm pvcreate /dev/sdc1
lvm pvcreate /dev/sdd1
```

Now `pvscan` will show new physical volumes.

```output
PV /dev/sdc1                      lvm2 [<7.28 TiB]
PV /dev/sdd1                      lvm2 [<7.28 TiB]
```

We will combine these two PVs into one single **virtual** group (VG). I named the volume group kepler_raid_vg0 but you can name it anything, for example raid0vg0.

```none
vgcreate kepler_raid_vg0 /dev/sdc1 /dev/sdd1
```

We can now see the volume group using `vgscan`.

```output
Found volume group "kepler_raid_vg0" using metadata type lvm2
```

Next we can create the **logical** volume (LV), it will be sized to use ALL the space in the VG.

* We use the --nosync flag to not sync these drives when the LV is created, any data written afterwards will be mirrored.
* We use the -l flag to set the size of the LV to 100% of the VG.
* We use the -n flag to name the LV to "kepler_raid_lv0".
* The last part `kepler_raid_vg0` is the name of the VG that we created before.

```none
lvcreate --mirrors 1 --type raid1 -l 100%FREE --nosync -n kepler_raid_lv0 kepler_raid_vg0
```

#### Creating FIle Systems

Use EXT4, it works best for LVM. I am making a simple EXT4 filesystem for this LV thats NOT encrypted.

```none
mkfs.ext4 /dev/kepler_raid_vg0/kepler_raid_lv0
```

To mount the file system we need the UUID of the file system we just created. Use `blkid`.

It will be called `/dev/mapper/*`. Since we are raiding the system, theres a couple to pick from.

All the UUIDs are the same, pick any.

```output
/dev/mapper/kepler_raid_vg0-kepler_raid_lv0_rimage_0: UUID="573ed9a5-1d18-43e4-abd9-92207a3dc6fd" BLOCK_SIZE="4096" TYPE="ext4"
/dev/mapper/kepler_raid_vg0-kepler_raid_lv0_rimage_1: UUID="573ed9a5-1d18-43e4-abd9-92207a3dc6fd" BLOCK_SIZE="4096" TYPE="ext4"
/dev/mapper/kepler_raid_vg0-kepler_raid_lv0: UUID="573ed9a5-1d18-43e4-abd9-92207a3dc6fd" BLOCK_SIZE="4096" TYPE="ext4"
```

Update /etc/fstab to mount the file system.

```none
sudo mkdir /mnt/kepler
```

**defaults** is specified so that we use the default mount settings (equivalent to rw,suid,dev,exec,auto,nouser,async).

I used 0 and 1 at the end of the map for these reasons.

* The 0 means to not dump the filesystem, dump is a linux tool for backing up the filesystem to another drive.
* The 2 is used by fsck to determine the order it should boot in, a root filesystem is always 1, and other filesystems should have the number.

```none
// /etc/fstab

UUID=573ed9a5-1d18-43e4-abd9-92207a3dc6fd /mnt/kepler ext4 defaults 0 2
```

To mount without rebooting run `sudo mount -a`.

### Monitoring The LVM Raid

Monitoring the status of raid can be done using the lvs tool. This will show the status of the raid: `lvs -a -o name,copy_percent,devices kepler_raid_vg0`.

```output
LV                         Cpy%Sync Devices/
kepler_raid_lv0            100.00   kepler_raid_lv0_rimage_0(0),kepler_raid_lv0_rimage_1(0)
[kepler_raid_lv0_rimage_0]          /dev/sdc1(1)
[kepler_raid_lv0_rimage_1]          /dev/sdd1(1)
[kepler_raid_lv0_rmeta_0]           /dev/sdc1(0)
[kepler_raid_lv0_rmeta_1]           /dev/sdd1(0)
```

We can get some more information about the devices in the raid with `sudo lvs --segments kepler_raid_vg0/kepler_raid_lv0 -o +devices`.

### Tearing Down an Raid Logical Volume

Use the lvconvert tool.

```none
sudo lvconvert --splitmirrors 1 --name copy kepler_raid_vg0/kepler_raid_lv0
```

Splits and creates these...

```output
sdc                                   8:32   0   7.3T  0 disk
└─sdc1                                8:33   0   7.3T  0 part
  └─kepler_raid_vg0-copy_raid_lv0 254:5    0   7.3T  0 lvm
sdd                                   8:48   0   7.3T  0 disk
└─sdd1                                8:49   0   7.3T  0 part
  └─kepler_raid_vg0-kepler          254:4    0   7.3T  0 lvm
```

We can rename the logical volume with `lvrename`.

```none
sudo lvrename kepler_raid_vg0 kepler_raid_lv0 arber_lv0
```

## LVM over MDADM

A different way of creating a raid is to use a specialized raid tool called mdadm.

We will use mdadm to create a raid 1 on the two drives, sdc and sdd. Then put LVM on top of that raid.

### Setting up MDADM

#### Partition Disks

This will wipe your drives.

Use pareted to create a GPT partition table, repeat for both drives.

```none
roland@store:~$ sudo parted -a optimal /dev/sdc
GNU Parted 3.4
Using /dev/sdd
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) mklabel gpt
(parted) mkpart primary 1 -1
(parted) name 1 raiddata1
(parted) set 1 raid on
(parted)

Information: You may need to update /etc/fstab.
```

Press `ctrl+d` to exit parted.

You can run `print` in parted to see something similar to this.

```output
(parted) print
Model: ATA ST8000VN004-2M21 (scsi)
Disk /dev/sdd: 8002GB
Sector size (logical/physical): 512B/4096B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name       Flags
 1      1049kB  8002GB  8002GB               raiddata1
```

Repeat this for SDC and SDD.

#### Configure MDADM

Create a new "array" in mdadm and put the two drives in it.

* --create specifies what block file will refer to this array. Pick an incrementing number is a good option starting at 0.
* --level is the raid level.
* --raid-devices is the number of devices in the array.

```none
mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdc1 /dev/sdd1
```

Create the file system.

```none
mkfs.ext4 /dev/md0
```

Check the status of the array.

```none
mdadm --detail /dev/md0
```

You can already read straight from `cat proc/mdstat` to see the status of the array as it builds.

Now we need to allow this array to be constructed on boot. Get the result of this command and paste it into the file `/etc/mdadm/mdadm.conf`.

```none
mdadm --detail --scan
```

```output
// /etc/mdadm/mdadm.conf
ARRAY /dev/md0 metadata=1.2 name=store:0 UUID=1991a11a:513644f9:fa014c72:783e3a00
```

Then update initramfs.

```none
update-initramfs -u
```

Next we need to mount this drive, do this through getting the UUID with `blkid` and pasting that into `/etc/fstab`.

```output
/dev/md0: UUID="0dcc2457-2ab4-45d9-afaf-49dd19c6acfe" BLOCK_SIZE="4096" TYPE="ext4"
```

```none
// /etc/fstab

...
UUID=0dcc2457-2ab4-45d9-afaf-49dd19c6acfe /mnt/arber ext4 defaults 0 2
```

Then we can mount the raid with `sudo mount -a` or `sudo mount /dev/md0`.

### How to Remove a Drive MDADM

Skip this step if you are following the instructions to create LVM on MDADM.

You need to fail the drive you are removing first so mdadm allows you to remove it.

First unmount your md0 drive thats mounted somewhere, for example `umount /mnt/arber`.

To mark the drive as failed run `mdadm /dev/md0 --fail /dev/sdd1`.

Then you can remove the drive with `mdadm /dev/md0 --remove /dev/sdd1`.

If you want to re add the drive run `mdadm /dev/md0 -add /dev/sdd1`. Of course it needs to be formatted correctly first, see above section (partition disks).

Now that /dev/sdd1 is removed, you CANNOT remount it, instead use mdadm to re assemble it and mount it through `/dev/md0`.

In this example `/dev/sdd1` is the drive that was removed.

```none
mdadm --stop /dev/md0
mdadm --assemble --run /dev/md0 /dev/sdd1
mount /mnt/md0 /mnt/recoveryFolder
```

### Starting and Stopping

To stop run `sudo mdadm --stop /dev/md0`.

To start run `sudo mdadm --assemble --run /dev/md0`.

### Solving md127

As far as i can tell, mdadm creates /dev/md127 when theres nothing in the config to read.

To fix this, check your `/etc/mdadm/mdadm.conf` and see if there is a md127 that you can change back to your prefferfed number.

Then run `update-initramfs -u`.

Then rebooting is the easiest way to fix it from here.

### Installing LVM on MDADM

Now that we have an MDADM array, we can install LVM on top of it.

First we will remove the filesystem we created earlier on /dev/md0 using `fdisk /dev/md0` and then writing with `w` to the device.

I had some trouble (i think caused by rebooting) so i had to run `mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdc1 /dev/sdd1` again.

Now that we have /dev/md0 with no filesystem, we should regenerate the config file with `mdadm --detail --scan`, replacing the old config with the new one.

This should output something like this into mdadm.conf.

```output
ARRAY /dev/md0 metadata=1.2 name=vm02:0 UUID=4a67294d:24b68bde:993efbda:f84261a7
```

Now follow LVM instructions.

```none
pvcreate /dev/md0
vgcreate arber_vg0 /dev/md0
lvcreate -n arber_lv0 -l 100%FREE arber_vg0
mkfs.ext4 /dev/mapper/arber_vg0-arber_lv0
```

Then use `blkid` to get the UUID of the new filesystem.

```output
/dev/mapper/arber_vg0-arber_lv0: UUID="b86caaff-5aec-42a4-88a9-9f2bd06c4cc8" BLOCK_SIZE="4096" TYPE="ext4"
```

Then add the new entry to `/etc/fstab`.

```none
UUID=b86caaff-5aec-42a4-88a9-9f2bd06c4cc8 /mnt/arber ext4 defaults 0 2
```

Then `mount -a` to mount the new filesystem. And you are done.
