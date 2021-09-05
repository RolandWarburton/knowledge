# vmnet Interfaces and Networking Explained

Between testing interfaces its important to flush your IP addresses using `sudo dhclient -r` to release and `sudo dhclient` to acquire a new one.

## VMNet0

VMNet0 is the **bridged** interface of the virtual machine.

*   VMNet0 is a virtual interface that exists on the host that VMWare machines can connect to.
*   Because the VM apears to be connected to the network directly it can receive an IP address from the DHCP server and connect to the outside world.
*   On a physical switch it sees both IP addresses and unique MACs for each (in the below screenshot, my physical PC and the VM also running on my PC using VMNet0 aka bridged mode)

![01](<assets/vmnet Interfaces Explained/01\_1.png>)
![02](<assets/vmnet Interfaces Explained/02\_1.png>)

![Source: Vision 6D](<assets/vmnet Interfaces Explained/Source: Vision 6D\_1.png>)

## VMNet1

VMNet0 is the host only network that isolates virtual machines from the rest of the network, other VMs, the internet, and anything in the outside world. No traffic goes in and no traffic goes out.

![03](<assets/vmnet Interfaces Explained/03\_1.png>)

## VMNet8

VMNet1 is the NAT interface of the virtual machine.

*   VMWare Workstation provides a virtual DHCP server and creates a NAT-ed IP address fot the VM to use
*   When the VM connects to the network its IP address will become a unique address picked by VMWare

![04](<assets/vmnet Interfaces Explained/04\_1.png>)
![05](<assets/vmnet Interfaces Explained/05\_1.png>)

## Custom networks

Lastly you can create your own custom interfaces by using lag segments.

![06](<assets/vmnet Interfaces Explained/06\_1.png>)
