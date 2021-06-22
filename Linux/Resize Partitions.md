# Resize Drives

## Shrinking a Filesystem and Partition

Take the following example.

```output
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   60G  0 disk
└─sda1   8:1    0   58G  0 part /
```

Lets say we want to shrink the /dev/sda1 partition to become 40G.

First we will need to shrink down the filesystem, then we will need to shrink down the partition. To do this we need to understand the difference between a **file system** and a **partition**. In my (possibly flawed) understanding here is a brief description of the two when related to resizing a drive.

in this situation a partition can be thought of as an entry in a table that describes where data should start and stop. IE that this "partition" begins at X and ends at Y. The partition does not care what goes inside its data container, that is the job of the filesystem.

A filesystem is something that goes inside a partition, a common filesystem that you might use is EXT4, the partition doesn't know where your files are, EXT4 does and as such we need to resize the filesystem first and squash the 58G of EXT4 filesystem occupied space down to 40G (assuming you have <40G of actual occupied filesystem space). Then after that our drive will have essentially partition space that used to be occupied by EXT4 that can now be shrunk down as well to the 40G.

Another way of thinking about partitions and filesystems is like a bucket.

* You have a bucket of water, your bucked can hold 10L of liquid (data). This is the total capacity of your bucket (drive)
* You draw fill lines on your bucket to arbitrarily separate the liquid (data) in the bucket, therefore you have partitioned your bucket
* You then fill the bucked with water (filesystem) and now no matter how full the bucket is, you will always know what liquid belongs to what partition of your bucket
* If you want to resize your bucket you can only do so at the end where the liquid meets the air (where your final partition meets free drive space)

```output
File system takes up the whole space of the partition
+------------------------+
|=====================|  |
+------------------------+

Step 1. Resize the file system with resize2fs
+------------------------+
|===========          |  |
+------------------------+

Step 2. Resize the partition
+------------------------+
|===========|            |
+------------------------+
```

### Shrinking Step 1

Booting into live media and using `fdisk /dev/sda` we can observe the information about the partition.

```output
Device    Boot    Start     End       Sectors    Size    Id    Type
dev/sda1  *       2048      121696863 121634816  58G     83    Linux
                  |          |
                  These will be important later
```

To shrink the drive all we need to do is these two commands within the live media on the **unmounted drives**.

```none
e2fsck -f /dev/sda1
resize2fs /dev/sda1 40G
```

Now our drive should be 40 Gigabytes, lets reboot and verify.

```none
df -h
```

```output
Filesystem    Size    Used    Avail    Use%    Mounted on
/dev/sda1     40G     5.5G    32G      15%     /
```

However we only resized the file size, not the partition. So when we inspect the drive in fdisk we get the same numbers.

```output
Device    Boot    Start     End       Sectors    Size    Id    Type
dev/sda1  *       2048      121696863 121634816  58G     83    Linux
                                                 |
                                        The size is still 58G.
                                        Because this is the partition size
                                        and we only resized the filesystem.
```

### Shrinking Step 2

Lets now reboot into the live media and resize the partition now.

To resize a partition we need the partition we are resizing to be the very last one on the disk, otherwise there is no way to resize it. So the first step is to use fdisk on /dev/sda to remove all partitions other than the main sda1.

Next write your changes to apply removing any partitions that come after the partition you wish to change.

From within the live media you can now use the `fdisk` tool to gather more information (or use the sector start and end you noted down before) to recreate the partition.

```none
fdisk /dev/sda

> print
Device    Boot    Start     End       Sectors    Size    Id    Type
dev/sda1  *       2048      121696863 121634816  58G     83    Linux

> d
Partition 1 has been deleted

> n
p <-- Create a primary partition
1 <-- Partition number will be 1
2048 <-- Enter the start sector you noted down before
+40.5G <-- Enter the relative end in Gigabytes, (i added 0.5 for a buffer)
N <-- We don't want to remove the Ext4 signature
```

Now when we run print in fdisk we should receive.

```none
Device    Boot    Start     End       Sectors    Size    Id    Type
/dev/sda1         2048      84912127 84910080    40.5G   83    Linux
```

Notice how the boot flag has been toggled off, we need to re-enable that.

```none
fdisk /dev/sda
> a <-- Toggle the boot flag
> w <-- Write your changes
```

## Expanding a Filesystem

Now that we have the theory and practice behind us with shrinking a drive, expanding a drive should be no problem (just do the opposite of above).

Again we MUST ensure that the partition we are working on is the final partition, if you need a refresher check the bucket example at the top of the page.

Our new partition table that we want to expand up to 50G. Using the `fdisk /dev/sda` `print` is noted below, notice how the actual **Disk** space is 60GiB, but the device **Partition** is only 40, that means we have room to expand back up and grow the disk.

```output
Disk /dev/sda: 60GiB, 64424509440 bytes, 125829120 sectors
Disk model: Virtual disk
Units: sectors of 1 * 512 = 512 bytes
...
Disklabel type: dos

Device    Boot    Start     End       Sectors    Size    Id    Type
/dev/sda1         2048      84912127 84910080    40.5G   83    Linux
```

```output
File system takes up the whole space of the partition
+------------------------+
|=========|              |
+------------------------+

Step 1. Resize the partition
+------------------------+
|===========          |  |
+------------------------+

Step 2. Resize the file system with resize2fs
+------------------------+
|=====================|  |
+------------------------+
```

### Growing Step 1

Using `fdisk` tool on `/dev/sda`

```none
sudo fdisk /dev/sda

> d
Partition 1 has been deleted

> n
p <-- Create a primary partition
1 <-- Partition number will be 1
2048 <-- Enter the start sector you noted down before
+50G <-- Enter the relative end in Gigabytes
N <-- We don't want to remove the Ext4 signature

w <-- Write the changes (will boot you out of fdisk)

sudo fdisk /dev/sda

> a <-- Toggle the boot flag
> w <-- Write your changes
```

### Growing Step 2

Now grow the file system to use the newly allocated partition size. Boot back into the live media to run this on the unmounted drive. There is no need to specify a second parameter in resize2fs because its implicitly will fill all available remaining space.

```none
e2fsck -f /dev/sda1
resize2fs /dev/sda1
```

And thats it! Reboot and see your newly resized file system.
