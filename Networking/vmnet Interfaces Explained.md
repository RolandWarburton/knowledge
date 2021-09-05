# vmnet Interfaces and Networking Explained

Between testing interfaces its important to flush your IP addresses using `sudo dhclient -r` to release and `sudo dhclient` to acquire a new one.

## VMNet0

VMNet0 is the **bridged** interface of the virtual machine.

* VMNet0 is a virtual interface that exists on the host that VMWare machines can connect to.
* Because the VM apears to be connected to the network directly it can receive an IP address from the DHCP server and connect to the outside world.
* On a physical switch it sees both IP addresses and unique MACs for each (in the below screenshot, my physical PC and the VM also running on my PC using VMNet0 aka bridged mode)

![01](https://i.imgur.com/nPKgTvd.png)
![02](https://i.imgur.com/i9uYsdt.png)

![Source: Vision 6D](https://i.imgur.com/6hxV3Di.png)

## VMNet1

VMNet0 is the host only network that isolates virtual machines from the rest of the network, other VMs, the internet, and anything in the outside world. No traffic goes in and no traffic goes out.

![03](https://i.imgur.com/rdznTJc.png)

## VMNet8

VMNet1 is the NAT interface of the virtual machine.

* VMWare Workstation provides a virtual DHCP server and creates a NAT-ed IP address fot the VM to use
* When the VM connects to the network its IP address will become a unique address picked by VMWare

![04](https://i.imgur.com/Lg7Br0N.png)
![05](https://i.imgur.com/y93Bc7V.png)

## Custom networks

Lastly you can create your own custom interfaces by using lag segments.

![06](https://i.imgur.com/PJISPyF.png)
