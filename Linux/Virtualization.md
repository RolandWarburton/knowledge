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

### Installing VMWare tools on Debian 10 (Buster)

Ensure that you have the CD/DVD hardware installed on your VM by right clicking on the vm in the left hand list of VMs and clicking on settings, then adding the CD/DVD drive if needed. Next make sure that you are pointing the CD/DVD drive to `/usr/lib/vmware/isoimages/linux.iso`.

Next, mount the drive and extract the installer.

```none
mkdir -p /mnt/cdrom
mount /dev/cdrom /mnt/cdrom
tar -zxpf /mnt/cdrom/VMwareTools-xx.y.zz-xxxxxx.tar.gz
```

Then navigate to the directory that VMwareTools extracted into (directory is called)

```none
cd vmware-rools-distrib/
```

Execute the installer

```none
sudo ./vmware-install.pl
```

Restart the VM and you are good to go!

#### open-vm-tools

In some cases open-vm-tools packages can be obtained from the OS vendor (eg debian) and can be installed from there. This is the preferred option if avaliable.

```none
sudo apt install open-vm-tools
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

2. Check which drive is the USB you are installing to with `lsblk`. In this case its `/dev/sdc`

3. partition the disk as a FAT32 drive

```none
sudo fdisk /dev/sdc
```

```none
"d" Delete all existing partitions (repeat until all are removed)
"n" Create 1 primary partition that extends the entire disk
"t" Set the type of drive to FAT32 (hex = b)
"a" Set the partition you just made to active
"p" Verify your partition table is correct
```

Example of correct partition table for USB stick

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

### Adding NFS datastores

This is a critical step for automating the deployment of new virtual machines.

First create a Network File Server on your host system. For arch linux users check [here](https://wiki.archlinux.org/index.php/NFS) for a great resource.

#### Creating a NFS server on your computer

In general follow these steps To create a file share to serve ISOs from.

In this example i am service Debian 10 Buster.

```none
sudo vim /etc/exports
```

```none
/srv/nfs/Debian10       192.168.0.110/24(no_subtree_check,rw,async)
```

Then mount the ISO and copy its contents into /srv/nfs/Debian10

```none
sudo mount ~/Debian10_Buster.iso ~/mount

sudo cp -r ~/mount/* /srv/nfs/Debian10
```

Then export your NFS

```none
exportfs -arv
```

Then start the NFS server and check its status

```none
sudo systemctl start nfs-server
sudo systemctl status nfs-server
```

```output
● nfs-server.service - NFS server and services
     Loaded: loaded (/usr/lib/systemd/system/nfs-server.service; disabled; vendor preset: disabled)
     Active: active (exited) since Thu 2020-07-09 02:06:27 AEST; 12min ago
    Process: 5265 ExecStartPre=/usr/sbin/exportfs -r (code=exited, status=0/SUCCESS)
    Process: 5266 ExecStart=/usr/sbin/rpc.nfsd (code=exited, status=0/SUCCESS)
   Main PID: 5266 (code=exited, status=0/SUCCESS)

Jul 09 02:06:27 Arch-Desktop systemd[1]: Starting NFS server and services...
Jul 09 02:06:27 Arch-Desktop systemd[1]: Finished NFS server and services.
```

#### Add the NFS on ESXi

In this example i am not using authentication and restriction to the NFS server should be done on an IP basis in `/etc/nfsconf`.

A datastore is created under **Storage -> Datastores -> New datastore**...

The ESXi settings are below worked for me:

```none
Name: Debian10
NFS Server: 192.168.0.100
NFS Share: /srv/nfs/Debian10
NFS Version: NFS 3
```

#### Connect ISO to new VM

Under Virtual Machines -> your virtual machine name. You can *Edit* the VM which will bring up a window that has 2 tabs (Virtual Hardware, and VM Options). Select Virtual Hardware -> CD/DVD Drive 1 and select the Datastore ISO file from the dropdown menu, make sure to connect the drive and power cycle the VM. You may need to do this a few times for it to register.

## ESXi Networking

These are some notes that i took when setting up a vlan for testing and development on my network.

#### Current network topology

My current setup contains:

1. PF box with 1 switch connected
2. The switch is uplinked to PF on an untagged port (a quirk with my edge switch prevents it from being a trunk i think)
3. The switch facilitates a tagged port (f0/3) to ESXi on vlan 100
4. The switch facilitates an untagged port (f0/1) to ESXi
5. The switch facilitates an untagged port to my PC

I have 2 networks:

1. My main network. Vlan 0 192.168.0.0/24
2. My net testing network. Vlan 100 192.168.100.0/24

#### Create port group in ESXi

This step is the easiest to do so we will do it first.

On ESXi, navigate to `Networking -> Port Groups` and click Add Port Group, specifying the VLAN ID.

Next ensure that your Guest VM is connected to the correct Port Group by going to the VM tab and "editing" the VMs settings to change the networking details.

#### Create my new vlan

On my PFSense router i have gone to `Interfaces -> Assignments -> Vlans` and created a new vlan with ID 100.

I then went to `Interfaces -> Assignments` and Created OPT1 interface on my lan parent LAN interface (in my case msk), so the new interface should be called **msk.100**. Make sure you are assigning the correct interface.

Next click on OPT1 to configure it.

1. Enable interface -> Checked
2. IPv4 Configuration Type -> Static IPV4
3. IPv4 Address -> 192.168.100.1/24
4. Do NOT assign a gateway to this interface

Next go to Services DHCP server and select the OPT1 tab for configuration. If you dont see OPT1 as an option you likely forgot to set the SN mask for OPT1 and it defaulted to /32 which is incorrect in this (and most) situations.

**Make sure to set the gateway, and be prepared to set the DG manually**. I have had some headaches caused by the DG not being set on the host by the DHCP server so try and manually specify it.

The DG IP will be the address of the interface. IE. OPT1 = 192.168.100.1 and my DG will be the same.

At this stage you should **check for connectivity**, however be prepared for it to fail. If it does continue onto the next steps.

#### Configure the switch to carry traffic for VL100

Next go to your switches interface. I have a Ubiquity EdgeMax that has a web interface.

Under the Vlans settings set the following. Be warned that coming from cisco IoS can throw you off as cisco calls its port types as *"Trunk"* and *"Access"* However other manufacturers call them untagged or tagged.

* Tagged = Trunked
* Untagged = Access

Use the picture for reference.

![Switch VL Settings](https://i.imgur.com/VPrdDtj.png)

#### Manually add firewall rules for OPT and LAN to communicate

At this step you should check if you can ping again before proceeding.

If ping still fails try looking at firewall rules next and adding allow rules where needed.

![LAN FW Rules](https://i.imgur.com/yNUA9bH.png)

![OPT FW Rules](https://i.imgur.com/gra9G4c.png)

#### Manually add the IP

At this step you should check if you can ping again before proceeding.

The next step is to look at the machine itself.

Start by looking at the current IP address of the machine using `ip a` or `ip addr`. I have found that all of my debian 10  machines have ens192 as their interface name.

The below is a working config for the ip settings, mostly just ensure that the machine has a correct IP and a correct subnet.

```output
2: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:5f:2d:09 brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.100/24 scope global ens192
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe5f:2d09/64 scope link
       valid_lft forever preferred_lft forever
```

If the information is incorrect issue a command to set the ip address, for example.

```none
ip addr add 192.168.100.100/24 dev ens192
```

#### Manually add Default gateway

The next Guest setting to check is the default gateway.

By using the `route` command (if you have it) you can see the known routes for the guest VM. If you dont have the route command you can also try `ip route` which will work in 99.99% of cases.

Heres an example of a working config.

```output
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown
172.18.0.0/16 dev br-582b597f70ba proto kernel scope link src 172.18.0.1
192.168.100.0/24 dev ens192 proto kernel scope link src 192.168.100.100
```

If you are missing a 192.168.100.x route then issue the following commands to add one.

Using the net-tools package which may or may not be avaliable to you.

```none
route add default gw 192.168.100.1 eth0
```

If you dont have net-tools you can always fall back on trusty ip.

```none
ip route add 192.168.100.1/24 dev eth0

OR a default gateway

ip route add default via 192.168.100.1 dev eth0
```

On an off note you can also re-route traffic this way.

```none
ip route add 192.168.100.1/24 via 192.168.0.1
```

Test your connection again. If you still dont have a connection then something is super wrong and i dont know how to fix it most likely, my inspiration for writing this is for self reference so i will add more issues and gotchas as i encounter them.

#### Some extra things to try

1. Can you ping from your PC, or your Router to the VM?
2. Can the VM ping anything? Check it can ping the router first on any gateway IP.
3. Run the packet capture debug tool on PF itself, or run it on your PC to look for clues
4. Use the traffic graphs to look for signs of life in PF

### Enabling passthrough for external storage (USB drives)

SSH into the controller and ensure that the USB arbitrator service is running.

Here i want to connect the LaCi drive in passthrough for a host, however the arbitrator service is disabled.

```none
esxcli hardware usb passthrough device list
```

```output
[root@lab-esxi-01:~] esxcli hardware usb passthrough device list
Bus  Dev  VendorId  ProductId  Enabled  Can Connect to VM                  Name
---  ---  --------  ---------  -------  ---------------------------------  ---------------------------------------------
1    3    4791      8064         false  no (usbarbitrator is not running)  Western Digital, G-Tech G-DRIVE mobile SSD
                                                                           R-Series
1    2    59f       105e         false  no (usbarbitrator is not running)  LaCie, Ltd
```

If its not running start it with the following command.

```none
/etc/init.d/usbarbitrator start
```

After that you might be prompted to restart hostd. Do that with the following command.

```none
/etc/init.d/hostd restart
```output
watchdog-hostd: Terminating watchdog process with PID 2098417
hostd stopped.
hostd started.
```

Then restart your UI (wait 1min and then refresh the page).

Before connecting the device you should shut down the VM if you can (avoid bad jujus). Now you should be able to select the vm and go to **edit** and then under the virtual hardware tab select "Add other device" and then "USB device". You should now see the new device on the hardware list. Make sure you select the correct device, they are labeled human friendly so in my case i used the drop down and selected the "LaCie Ltd" drive from the list.

i did have some issues with the passthrough, sometimes its a good idea to create a whole new VM and test until it works. I know for sure that the above notes work on a fresh vm with the following settings:

* USB Controller: USB 3.0 (despite the not supported warning)
* Only 1 USB device
* Tested on Debian 10 Buster

### ESXi USB Passthrough debugging

SSH into the ESXi Host. Run `esxcli hardware usb passthrough device list` to list the connected external drives. The idea of this is to pass an external USB drive straight through to a particular VM so that it can be mounted.

```output
Bus  Dev  VendorId  ProductId  Enabled  Can Connect to VM  Name
---  ---  --------  ---------  -------  -----------------  -------------------------------------------------------------
1    2    4791      8064          true  yes                Western Digital, G-Tech G-DRIVE mobile SSD R-Series
1    3    59f       105e          true  yes                LaCie, Ltd
```

I believe the device type of my external drive is NTFS. This is confirmed with `lsblk /dev/sdb2 --output-all`.

```output
roland@debian:~$ `mount | grep "^/dev"`
/dev/sdb2 on /home/sftp/lacie type fuseblk (rw,relatime,user_id=0,group_id=0,allow_other,blksize=4096)
```

The drive is formatted like so

```output
sdb      8:16   0  2.7T  0 disk
├─sdb1   8:17   0  200M  0 part
└─sdb2   8:18   0  2.7T  0 part
```

Make sure to start the USB arbitrator with the following command... Optionally you can enable it as well.

```none
/etc/init.d/usbarbitrator start
/etc/init.d/usbarbitrator enable
```

Then restart hostd.

```none
/etc/init.d/hostd restart
```

To debug a drive that was once working (showing up) and is now dead (orphaned) after an unexpected reboot or power outage try these following commands.

Check the dead mpaths. [Source](https://kb.vmware.com/s/article/2145752). If you see a dead drive reported then the easiest option is to reboot.

```none
esxcfg-mpath -L | grep dead
```

After rebooting the host itself, make sure to verify the settings of the virtual machine, i selected a USB controller for USB 3.0 and made sure that i had added a new USB drive device added under `virtual hardware > add another device`.
