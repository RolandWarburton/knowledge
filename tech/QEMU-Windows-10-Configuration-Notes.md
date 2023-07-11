# QEMU Windows 10 Configuration Notes

For a more in-depth and general overview, i have nodes on QEMU in Linux/Virtulization.md
within this repository on github (rolandwarburton/knowledge)

For situations that require windows, such as building & packaging software
QEMU/KVM provides a great way to spin up a windows 10 machine.

While QEMU is a simple answer for hosting a windows virtual machine.
Sharing files between the host and guest requires evaluating different approaches.
For this one i will cover an RDC based file share. RDC (remote desktop connection)
is a part of RDP (remote desktop protocol) that Microsoft offers for windows.

To connect to the RDP session and to create a shared folder i use Remmina, a remote desktop client.

Make sure you have the following dependencies first.

```none
sudo apt install qemu-kvm libvirt-clients libvirt-daemon-system virtinst bridge-utils
```

## Method

Create a virtual drive.

```none
sudo qemu-img create -f qcow2 /var/lib/libvirt/images/win10.qcow2 20G
```

Create a `domain.xml` file that describes how the VM will be created.

```html
<domain type='kvm'>
  <name>win10</name>
  <memory unit='KiB'>4194304</memory>
  <vcpu placement='static'>2</vcpu>
  <os>
    <type arch='x86_64' machine='pc-q35-8.0'>hvm</type>
    <boot dev='cdrom'/>
    <boot dev='hd'/>
  </os>
  <features>
    <acpi/>
    <apic/>
  </features>
  <cpu mode='host-passthrough' check='none' migratable='on'/>
  <clock offset='localtime'>
    <timer name='rtc' tickpolicy='catchup'/>
    <timer name='pit' tickpolicy='delay'/>
    <timer name='hpet' present='no'/>
    <timer name='hypervclock' present='yes'/>
  </clock>
  <on_poweroff>destroy</on_poweroff>
  <on_reboot>destroy</on_reboot>
  <on_crash>destroy</on_crash>
  <pm>
    <suspend-to-mem enabled='no'/>
    <suspend-to-disk enabled='no'/>
  </pm>
  <devices>
    <emulator>/usr/bin/qemu-system-x86_64</emulator>
    <!-- file system -->
    <disk type='file' device='disk'>
      <driver name='qemu' type='qcow2' discard='unmap'/>
      <source file='/var/lib/libvirt/images/win10.qcow2' index='2'/>
      <backingStore/>
      <target dev='sda' bus='sata'/>
      <alias name='sata0-0-0'/>
      <address type='drive' controller='0' bus='0' target='0' unit='0'/>
    </disk>
    <!-- iso mount  -->
    <disk type='file' device='cdrom'>
      <driver name='qemu' type='raw'/>
      <source file='/home/roland/Downloads/en-us_windows_10_22h2_x64.iso' index='1'/>
      <backingStore/>
      <target dev='sdb' bus='sata'/>
      <readonly/>
      <alias name='sata0-0-1'/>
      <address type='drive' controller='0' bus='0' target='0' unit='1'/>
    </disk>
    <!-- networking -->
    <interface type='network'>
      <mac address='52:54:00:f1:86:21'/>
      <source network='default' portid='20cb4a90-36b7-4b7e-996f-ca261cd66050' bridge='virbr0'/>
      <target dev='vnet11'/>
      <model type='e1000e'/>
      <alias name='net0'/>
      <address type='pci' domain='0x0000' bus='0x01' slot='0x00' function='0x0'/>
    </interface>
    <!-- input -->
    <input type='mouse' bus='ps2'>
      <alias name='input1'/>
    </input>
    <input type='keyboard' bus='ps2'>
      <alias name='input2'/>
    </input>
    <!-- video -->
    <graphics type='spice' port='5901' autoport='yes' listen='127.0.0.1'>
      <listen type='address' address='127.0.0.1'/>
      <image compression='off'/>
    </graphics>
    <!-- virtual gpu -->
    <video>
      <model type='qxl' ram='65536' vram='65536' vgamem='16384' heads='1' primary='yes'/>
      <alias name='video0'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x01' function='0x0'/>
    </video>
    <!-- audio -->
    <sound model='ich9'>
      <alias name='sound0'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x1b' function='0x0'/>
    </sound>
    <audio id='1' type='spice'/>
    <!-- dynamic memory allocation from the host -->
    <memballoon model='virtio'>
      <alias name='balloon0'/>
      <address type='pci' domain='0x0000' bus='0x04' slot='0x00' function='0x0'/>
    </memballoon>
  </devices>
</domain>
```

Start the network.

```none
sudo virsh net-start default
```

Start the virtual machine

```bash
export MACHINE_NAME='win10'
virsh -c qemu:///system define domain.xml
virsh -c qemu:///system start $MACHINE_NAME
```

Next you need to install Remmina and create a new connection profile.

* Name: win10 dev box
* Username: USERNAME
* Password: PASSWORD
* Share Folder: /home/roland/share

Your share drive should appear under "restricted drives and folders".

## Accessing The Shared Drive

Since our drive is not "real" it can be hard to target files in it,
for example running the `cd` command on `\\tsclient\_home_roland_share`
would not be supported because it is a UNC path.

To resolve this you can assign a drive letter to the UNC path instead,
Allowing for command line access.

```none
net use "Z:" "\\tsclient\_home_roland_share"
```

You can then change directory and run your program.

```none
Z:
myProgram.exe --flag value
```
