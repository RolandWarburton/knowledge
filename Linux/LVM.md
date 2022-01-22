# LVM Notes

Traditionally without LVM you have three layers of storage:

1. Hard drive: `/dev/sda`
2. Partitions: `/dev/sda1`
3. File system: `/dev/sda1` formatted as EXT4 mounted at `/`

LVM add three layers between the **Partitions** and the **File system**.

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
