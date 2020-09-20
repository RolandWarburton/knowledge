# Networking (Part Two). Network File Systems (NFS)

NFS allows you to create shared directories and do other stuff.

### Create a shared directory

1. Create a folder in `/srv/nfs` on the host machine `mkdir -p /srv/nfs/test /mnt/test`. Make sure to `chown` anything inside the `nfs/*` folder with a user that isnt root otherwise you will be unnable to write anything on the client side.
2. edit `/etc/exports` and add a line to export this file when you host a NFS server.

```none
#/etc/exports
/srv/nfs        192.168.0.0/24(no_subtree_check,rw,sync,crossmnt,fsid=0)
/srv/nfs/test	  192.168.0.0/24(no_subtree_check,rw,sync)
```

3. Then on your Client machine mount the NFS with `sudo mount HOST_IP:/srv/nfs /mnt/`

### Network Boot live Arch ISO (PXE)

The instructions on the [wiki](https://wiki.archlinux.org/index.php/PXE#Network) for this are confusing for noob users like myself. Here are my instructions.

To network boot you will need

1. A copy of the Arch ISO
2. A DHCP and TFTP server `dnsmasq`
3. HTTP server to serve the ISO `darkhttpd`
4. A Network File System (NFS). This already comes with Arch.

1. Make sure your host has a static IP. My host's IP is 192.168.0.10 (via `hostname -i`)

2. Per the instructions on the wiki. Mount the Arch ISO so its mounted file structure can be later copied to another location to serve it from.

```none
mkdir -p /mnt/archiso
sudo mount -o loop,ro archlinux-2020.01.01-x86_64.iso /mnt/archiso
sudo mv -r /mnt/archiso/ /srv/pxearch
```

3. Using dnsmasq allows you to configure the networking (DHCP) for the client (darkhttpd actually serves the files) and facilitate files being transfered between the PXE host and client. **Dont forget to start** and check the status of `dnsmasq.service` when debugging.

```none
# Config on arch wiki but slightly modified
# /etc/dnsmasq.conf
port=0
interface=enp6s0
bind-interfaces
# This is the range of IP addresses that devices will get
dhcp-range=192.168.0.50,192.168.0.150,12h
# These files are relative to the location of the tftp-root (tftp is chrooted)
dhcp-boot=/arch/boot/syslinux/lpxelinux.0
dhcp-option-force=209,boot/syslinux/archiso.cfg
dhcp-option-force=210,/arch/
# This is my router
dhcp-option-force=66,192.168.0.1
enable-tftp
# this is where we are serving the arch base files from.
tftp-root=/srv/pxearch
```

4. For TFTP to work your system needs to know to *export* the location of the filesystem (/srv/pxearch).

* re-export these new settings with `exportfs -rav`
* Start the nfs server `sudo systemctl start nfs-server`

```none
/etc/exports
/srv/pxearch  192.168.0.0/24(ro,no_subtree_check)
```

To transfer the filesystem from `/srv/pxearch` (or any other alt location) over you should use darkhttpd to transfer the file over http.
Start by running `sudo darkhttpd /srv/pxearch`

### Missing pxelinux.0 file error

if you run into an pxelinux error, it can be identified by looking at `journalctl -u dnsmasq`.
To solve this copy the file to its expected location `sudo cp /mnt/archiso/arch/boot/syslinux/lpxelinux.0 /srv/pxelinux/pxelinux.0`

### Chroot nfs-server share

Instructions for the server...

```none
vim /etc/exports
```

```none
/mnt/lacie 10.10.10.0/24(rw,sync,subtree_check)
```

---

```none
useradd -g sftp_users -s /bin/false -m -d /home/sftp sftp
passwd sftp
```

```none
chown root: /home/sftp
sudo chown root: /home/sftp
```

```none
mkdir /home/sftp/test
chmod 755 /home/sftp/test
chown sftp:sftp_users /home/sftp/test
```

```none
vim /etc/ssh/sshd_config
```

```output
# CHANGE
Subsystem sftp /usr/lib/openssh/sftp-server
# TO
Subsystem sftp internal-sftp
```

Then add a chroot for sftp_users.

```none
Match Group sftp_users
	ChrootDirectory %h
	ForceCommand internal-sftp
	AllowTcpForwarding no
	X11Forwarding no
```

```none
systemctl restart sshd
systemctl status sshd
```
