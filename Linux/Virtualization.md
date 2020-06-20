# Virtualization

So far i havent tried out much virtulization on linux, i plan on experiemnting with esxi 6.5 when i get some hardware that can run it however 🙂.

## VMware

### Fix internet issues

You can review what network adaptors vmware will look for by running `/usr/bin/vmware-netcfg` and typing your password when prompted.

Make sure you have started the networking services.

```none
sudo systemctl start vmware-networks.service
sudo systemctl enable vmware-networks.service
```

Then "redetect" your network devices

```none
sudo modprobe vmnet && sudo vmware-networks --start
```

### vmmon issues

Try following these instructions.

```none
==> Before using VMware, you need to reboot or load vmw_vmci and vmmon kernel modules (in a terminal on root: modprobe -a vmw_vmci vmmon)
==> You may also need to enable some of these services:
- vmware-networks.service: to have network access inside VMs
- vmware-usbarbitrator.service: to connect USB devices inside VMs
- vmware-hostd.service: to share VMs on the network
```

```none
sudo modprobe -a vmw_vmci vmmon
sudo systemctl start vmware-networks.service
sudo systemctl start vmware-usbarbitrator.service
sudo systemctl start vmware-hostd.service
```

## ESXI 6.5

### Installation

#### Prepare the USB as a FAT32 drive

[source](https://docs.vmware.com/en/VMware-vSphere/6.7/vsphere-esxi-67-installation-setup-guide.pdf).

1. Download required tools

```none
sudo pacman -S mtools
sudo pacman -S syslinux

```

2. Check which drive is the USB you are installing to witn `lsblk`. In this case its `/dev/sdc`

3. partition the disk as a FAT32 drive

```none
sudo fdisk /dev/sdc
```

```none
"d" Delete all existing partitions (repeat until all are removed)
"n" Create 1 primary partition that extends the entire disk
"t" Set the type of drive to FAT32 (hex = b)
"a" Set the parition you just made to active
"p" Verify your partition table is correct
```

Example of correct parition table for USB stick

```none
Disk /dev/sdc: 14.45 GiB, 15502147584 bytes, 30277632 sectors
Disk model: DataTraveler 3.0
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xe8b975ed
```

4. Load MBR with syslinux

```none
sudo syslinux /dev/sdc1
```

5. Mount the esxi iso

```none
sudo mount ~/Documents/ISOs/VMware-VMvisor-Installer-201908001-14320405.x86_64.iso ~/mount/
```

6. Mount the USB iso

Run the below command or click on the drive in a file manager like thunar or dolphin to mount the usb drive.

```none
sudo mount /dev/sdc /run/media/roland/USB
```

7. Copy the esxi ISO to the USB

```none
cp -r -V ~/mount/* /run/media/roland/USB/
```

1. Rename isolinux to syslinux

```none
mv /run/media/roland/USB/usbdisk/isolinux.cfg /run/media/roland/USB/syslinux.cfg
```

9. Edit sysl

```none
vim /run/media/roland/USB/syslinux.cfg
(edit so it says "APPEND -c boot.cfg -p")
```

10. Unmount the two drives

```none
sudo umount ~/mount
sudo umount /run/media/roland/USB
```
