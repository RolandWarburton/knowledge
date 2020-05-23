# PFSense on a laptop

![pktTracer_Diagram](/media/PFSense_Diagram.png)

## Installing
1. Go to [the website](https://www.pfsense.org/download/) and download the latest version
```
Version: 2.4.5
Architecture: AMD64 (64bit)
Installer: USB Memstick Installer
Console: VGA
Mirror: Singapore
```
2. Flash image to usb `sudo dd status=progress if=pfSense.img of=/dev/sd[drive]`
3. Follow the install instructions in the installer
4. If you are installing on a laptop (like me) you will likely only have 1 eth port on your laptop, so you will need a switch to facilitate your internet modem, computer and other devices, and your laptop pfsense router. I am using a Cisco 3750 and have connected F0/0/1 -> modem, and F0/0/2 -> laptop router.

## Setting up Basic Configuration
* My current router is on 192.168.0.0/24
* ISP/default gateway (telstra modem) = 192.168.0.1
* Switch LAN IP = 192.168.0.10
* Switch LAN vlan ID = 200 
* PFSense WAN = re0.100	no ip
* PFSense LAN = re0.200 192.168.0.200

#### Switch Config
```
vlan 200
	name LAN

int vlan 200
	ip add 192.168.0.10 255.255.255.0

int f1/0/1
	description connection to internet
	switchport mode access
	switchport access vlan 200

int f1/0/3
	description connection to PFSense
	switchport trunk encapsulation dot1q
	switchport mode trunk

! An example interface that you would put a LAN PC on that reports to PFSense
int f1/0/3
	description "put a LAN PC here"
	switchport access vlan 200
	switchport mode access
```

#### PFSense Config
1. Create Vlans -> yes
2. Create 2 vlans on your single internet for WAN and LAN
   1. re0.100 = WAN
   2. re0.200 = LAN

Proceed and have PFSense finish setting up the router, you will receive the following IPs
```
WAN (wan)	-> re0.100	-> 
LAN (lan)	-> re0.200	-> 192.168.1.1
```

Select option 1 to **assign interfaces** and change re0.200 to an IP that exists on your current network. My current network is using 192.168.0.0/24 so i will change the IP to 192.168.0.200 which will become the new gateway and ip to access PFSense.

```
WAN (wan)	-> re0.100	-> 
LAN (lan)	-> re0.200	-> 192.168.0.200
```

You should now be able to connect to 192.168.0.200 (pfsense) from a computer on the 192.168.0.0/24 network. Once logged in go through the setup process and set there values
* Primary DNS = 8.8.8.8
* Secondary DNS = 8.8.4.4
* Enable internal traffic, because the PFSense box is on the inside of the network

#### Testing
* You should be able to ping 192.168.0.10 (switch) -> 192.168.0.1 (modem)
* 192.168.0.10 (switch) -> 192.168.0.200 (pfsense) and vice versa
* You should be able to access 192.168.0.200 (pfsense) from a computer connected anywhere on 192.168.0.200

#### Setting up SSH to connect to switch
```
username roland privilege 15 secret p@ssw0rd
ip domain-name Sw4.com
crypto key generate rsa modulus 2048
ip ssh version 2
line vty 0 15
	transport input ssh
	login local
```

#### Setting up SSH to connect to PFSense
web interface -> system -> advanced -> Secure Shell
* Enable Secure Shell = true
* Allow Agent Forwarding = true
* SSHd Key Only = Password or Public Key
