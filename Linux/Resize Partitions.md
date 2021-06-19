# Resize Drives

## Shrinking a Filesystem and Partition

Take the following example.

```output
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   60G  0 disk
└─sda1   8:1    0   58G  0 part /
```

Lets say we want to shrink the /dev/sda1 partition to become 40G.

First we will need to shrink down the filesystem, then we will need to shrink down the partition. To do this we need to work through two steps and understand the difference between a file system and a partition. In my (possibly flawed) understanding here is a brief description of what we will be doing.

in this situation a partition can be thought of as an entry that sits in a table on a drive that describes a "shell". IE that this "partition" begins at X and ends at Y. The partition does not care what goes inside its shell, that is the job of the filesystem.

A filesystem is something that goes inside a partition, a common filesystem that you might use is EXT4, the partition doesn't know where your files are, EXT4 does and as such we need to resize the filesystem first and squash the 58G of EXT4 filesystem occupied space down to 40G (assuming you have <40G of actual files). Then after that our drive will have essentially partition space that used to be occupied by EXT4 that can now be shrunk down as well to the 40G.

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
dev/sda1          2048      84912127 84910080    40.5G   83    Linux
```

Notice how the boot flag has been toggled off, we need to re-enable that.

```none
fdisk /dev/sda
> a <-- Toggle the boot flag
> w <-- Write your changes
```
