# Managing Disks

## Permanent Disk Mapping with UDEV

Get the device details, then create ``

```bash
sudo udevadm info /dev/sda | grep ID_SERIAL=

# output
ID_SERIAL=Samsung_SSD_870_EVO_1TB_ABCDEFG123

```

Write a rule for this drive.

```bash
# /etc/udev/rules.d/99-drive-names.rules
KERNEL=="sd*", SUBSYSTEM=="block", ENV{ID_SERIAL}=="Samsung_SSD_870_EVO_1TB_ABCDEFG123", SYMLINK+="virtual-machines"
```

Then reload the rules.

```bash
sudo udevadm control –reload-rules

sudo udevadm trigger
```
