# Managing Drives in Linux

How to mount EXT and NTFS drives

#### EXT file systems

1. Check for the drive `lsblk`
2. Mount the drive in the users folder (create/use the 'media' folder)\
`sudo mount /dev/sdb2 /home/roland/mount`
3. Give the user permission
`sudo chown roland /home/roland/mount`

Unmount with `sudo umount /home/roland/Media`.

#### NTFS (Windows) file systems

install the [ntfs-3g](https://wiki.archlinux.org/index.php/NTFS-3G) package.\
Mount the NTFS drive with `sudo sudo ntfs-3g /dev/sdb2 /home/roland/mount`.\
Unmount the drive the same way you would with any filesystem `sudo umount /home/roland/Media`.

### Automatic mounting

You can install [thunar-volman](https://www.archlinux.org/packages/extra/x86_64/thunar-volman/) to automatically mount a volume when its plugged in.
