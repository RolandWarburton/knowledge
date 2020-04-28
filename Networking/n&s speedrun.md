# N&S Speedrun

This doc covers the entire Networks and Switching introductory unit as fast as possible. This is great for getting the bare minimum down or for use as a reference. However i recommend putting in some effort in practicing the concepts and labs to avoid choking on the practical exam.

This guide was written based on the syllabus for Networks and Switching in Semester 1 2019, though it will likely stay relevant for many years to come as the core concepts dont change that much, however given the recent changes to CCNA1 some additional concepts might be in your syllabus (2020 S2 onwards).

This whole guide should take ~15mins to read carefully and perhaps a day or 2 to practice and learn.

Without further note, lets get started!

## Core concepts

#### Theory
* Subnets
* L2 vs L3 switching
* Binary
* Switches
* Routers

#### Configuration
* VLANs
* SSH
* Router on a stick
* Switchport security
* Etherchannel
* Spanning tree
* BPDU guard
* Portfast

#### Preparing for assessments
* Debugging
* Midterm exam
* Gotchas
* Final skills exam questions / test bank
* Case study


## Theory

This should account for around 15% of your knowledge/effort put into the unit. However if you still do not understand VLSM then it may be worth your time to grind out this key point of knowledge

### Subnets

You should understand these concepts
* How to subnet (VLSM)
* Describe subnet classes (classful subnetting)

#### VLSM

There are many ways to do subnetting. If you already have a way that works, dont relearn it. Stick with what makes sense to you.

Heres what makes sense to me.

![CIDR](/media/CIDR.png)

![CIDR_mini](/media/CIDR_mini.png)

### L2 vs L3 switching

You should understand the difference and applications of L2/L3 switching.

The main difference is L2 cannot change a packets VLAN and L3 can. The application of a L2 switch is for cost effective deployments, and lower overhead. The application of a L3 switch is to be used either as a core or distribution layer switch (depending on the network size).

#### Layer 2 Switching

L2 switching is the most common type of switching within this unit. However its becoming less and less common in real life due to cost reductions of L3 switching.

#### What is L2 switching?

L2 switching is switching performed on L2 of the OSI model (data link).

L2 switching limits the manipulation that a switch can perform on a packet. This limitation prevents a switch from changing the packets VLAN. Typically switching the packets VLAN is performed on the router.

#### Layer 3 Switching

L3 switching is performed on L3 of the OSI model (Network layer)

L3 switching is a more performance (computationally) intensive task where the packet has to be de-encapsulated more that L2 to have its VLAN re-tagged. 

The most important thing to note is that there is a penalty in this unit for using a L3 switch on your mid semester exam as sometimes Jason will provision a L3 switch in your exam when most students are used to L2 switches. You will know if this is the case (given the hardware hasn't changed) in ATCs labs because your switches interfaces will be 3 numbers long, for example `f0/0/0`.

### Binary

If you are a CS student you are likely doing CLE at the same time and know what binary is so ill skip it.

You should understand binary while you are learning subnetting on a theory level only. If you are referring to my preferred method of subnetting with a pen and paper than you dont need it. 

If you are using a different technique you may need to have to convert between up to 8 bits of binary to decimal for each octet of your IPv4 address. Make sure to include these tables in your bound reference book. It covers enough of the conversions that you can manually add or subtract any missing bits without making a fatal math error.


| Bin       | Dec |
|-----------|-----|
| 100000000 | 128 |
| 11000000  | 192 |
| 11100000  | 224 |
| 11110000  | 240 |
| 11111000  | 248 |
| 11111100  | 252 |
| 11111110  | 254 |
| 11111111  | 255 |



| Bin       | Dec |
|-----------|-----|
| 011111111 | 127 |
| 00111111  | 63  |
| 00011111  | 31  |
| 00001111  | 15  |
| 00000111  | 7   |
| 00000011  | 3   |
| 00000001  | 1   |
| 00000000  | 0   |



| Bit | Val |
|-----|-----|
| 1   | 1   |
| 2   | 2   |
| 3   | 4   |
| 4   | 8   |
| 5   | 16  |
| 6   | 32  |
| 7   | 64  |
| 8   | 128 |

### Hardware

To understand this concept you should be able to explain the purpose of a switch and a router. This section shouldn't take too much time or effort.

#### What is a switch?

A switch connects computers together on an **internal** network by literally switching a packet from one computer to another.

Switching happens over one network only.

A network should be considered as either:
* A single VLAN - A packet cannot be *tagged* for another vlan on a L2 switch (L2 switches are covered in Theory)
* An uninterrupted portion of a network where a packet can travel from computer A to B without needing to cross over a router or change vlan

Furthermore on networks. There are two types of networks that you should be aware of for the final exam. Logical and physical LANs. In the above list i say that a network can be defined within a VLAN, or virtual LAN. Compare this to a physical LANs where a *network* is separated by a router.

#### What is the purpose of using a switch?

> The purpose of a switch is to connect multiple networked devices together over a physical medium

#### Types of switches
* Layer 2 (L2) switches
* Layer 3 (L3) switches
* Frame relay switches (not covered in this unit but you should be aware of their existence)

#### What is a router?

A router directs traffic from different networks within your internal network.
* A router generally sits on the edge of a network and connects to the internet (this is the only type you will encounter in n&s)
* A router may also NOT sit on the edge of your network and route traffic from internal network A to internal network B (this is not a focus of n&s)


## Configuration

This should account for around 70% of your knowledge/effort put into the unit. The best way to approach the labs are by concept, not by week or lab number. Luckily each lab tends to focus on a single topic so covering these is pretty straight forward.

In (almost) no particular order here are the notes!

### VLANs

A VLAN is short for virtual LAN (see *what is a switch* for a bit more about VLANs). VLANS are **only** applied to a switch. A VLAN virtually separates your switch into networks.

Every switch has a default vlan (VLAN 1). VLAN 1 has **every** port assigned to it by default. This implies that you can assign the physical ports to VLANS which is covered in a min.

#### VLAN bare bones config

![simpleVLAN](/media/simpleVLAN.png)

```
! Select VLAN 10
S1(config)#vlan 10
! Name VLAN 10 test
S1(config-vlan)#name test
S1(config-vlan)#int vlan 1
S1(config-if)#ip add 192.168.0.1 255.255.255.0
S1(config-if)#no shut
```

```
! PC 1
!
! PC 1 has to have an ip within the 192.168.0.0/24 subnet
! So any address between 192.168.0.1 and 192.168.0.254
! The IP cannot be 192.168.0.0 or 192.168.0.255
! These are reserved for the network ID and broadcast address
ip address: 192.168.0.10 255.255.255.0

! PC 1
default gateway: 192.168.0.1
```

### SSH

SSH uses RSA keys to authenticate a user (usually from a computer) to a switch or router.

![PT_SSH_Diagram](/media/PT_SSH_Diagram.png)

#### Create a user on a switch to connect to from a remote location
```
! Create a user from S1 to access from remote locations
S1(config)#username [name] privilege 15 secret [pass]
S1(config)#ip domain-name [domain.com]

! Generate an RSA key for your user to present when authenticating
! Aim for >1024 for SSH V2
S1(config)#crypto key generate rsa

! Enable SSH on vty (virtual terminal, ie logging in remotely)
S1(config)#line vty 0 15
! Set vty to accept ssh
S1(config-line)#transport input ssh
! use the credentials on this LOCAL machine to authenticate
S1(config-line)#login local

! Enable SSH V2 (optional but recommended)
S1(config)#ip ssh version 2
```

#### Connect to S1 from S2

Since we enabled login local on S1 we dont need to copy keys over (we just use the password that we created on S1). Yes this does invalidate the security of RSA keys because we are back to relying on human readable passphrases but thats besides the point in this unit.

From PC1 open up the SSH connection screen and type your username and password, it should connect.

![PT_SSH_Icon](/media/PT_SSH_Icon.png)

#### Managing SSH

* Delete a user with `no username [name]`
* Change the SSH timeout `ip ssh timeout [seconds]`
* limit auth retries `ip ssh authentication-retries`

To remove SSH:
```
line vty 0 15
transport input none
no login
```

### Telnet

Telnet is SSHs keyless and exploitable (but easier to implement) brother.

```
line vty 0 15
password [pass]
```

Then access telnet through the PC.

### Router on a stick

ROS is one of the most important concepts in N&S.

* ROS is **literally** just the .x on a router interface. For example `g0/0/1.10`.
* After a `.10` **sub interface** has been applied, the routers physical `0/0/1` port is considered virtually split. 
* The `0/0/1.10` **sub interface** of `0/0/1` can now handle traffic from VLAN 10.
* You can repeat this for every vlan you have, including the default VLAN 1 (`0/0/1.1`). 
This removes the need to have a separate port for each VLAN.

![ROS](/media/ROS.png)

#### Switch
1. Create vlan 10
2. The ip of VLAN 10 can be any internal IP - `192.168.0.2 255.255.255.0`
3. Make sure the router side interface is trunking
4. Make the the PC side interface is on vlan 10 - `switchport access vlan 10`
5. The default gateway of the switch is the router - `ip default-gateway 192.168.0.1`

#### Router
1. Create subinterface 10 - int g0/0/1.10
2. Set subinterface 10 to dot1Q [vlan number] - `encapsulation dot1q 10`
3. Set the IP of the subinterface to an address in VLAN 10 - `ip add 192.168.0.1 255.255.255.0`

#### PC
1. The ip of the PC is an IP in the VLAN 10 range - `192.168.0.10/24`
2. The default gateway is the IP of the **router** because the router is the exit point to the VLAN 10 network

### Switchport security

Switchport security is a list of sub menus.

A common gotcha is setting your port security before you configure the rest of your network. **switchport security should be configured last**.

To configure port security, apply these commands to an interface - `int f0/x` or select a range of ports with `int range f0/x-y`.

#### Set max MAC addresses
```
switchport port-security maximum [number]
```

Then configure a violation when the max MACs are exceeded.
```
switchport port-security violation [restrict/protect/sticky]
! restrict - drop unknown macs and log it
! protect - drop unknown macs and dont log it
! sticky - the mac ports can never change and once set can not be forgotten
```

#### Allow specific MAC
```
switchport port-security mac-address xx-xx-xx-xx-xx-xx
```

#### Check your port security
```
show port security

! if you are in config mode you can run
do show port security
```

#### Enable console cable security
```
! Create a user to authenticate with when logging in with
username [name] privilege 15 secret [pass]

! Enable logins for users over con (console)
line con 0
login local
```

### Etherchannel

There are 2 types of etherchannel bundling. PAGP and LACP.

The only difference between PAGP and LACP is the key words used to define configure the switch that is in charge of the link.

* LACP - desirable and auto
* PAGP - active and passive

#### PAGP

![PAGP](/media/PAGP.png)

```
! Switch 1
int range f0/1-2
switchport mode trunk
channel group 1 mode desirable

! Switch 2
int range f0/1-2
switchport mode trunk
channel group 1 mode auto
```


#### LACP

![LACP](/media/LACP.png)

```
! Switch 1
int range f0/1-2
switchport mode trunk
channel group 1 mode active

! Switch 2
int range f0/1-2
switchport mode trunk
channel group 1 mode passive
```

#### 3 Switch etherchannel

Etherchannel over 3 switches is exactly the same. Just make sure that you pick different group for each link within the network to avoid conflicts.

1. Each link between switch A and B are **trunking**
2. Each link on switch A and B are in the **same etherchannel group**
3. A 3 switch etherchannel with have 2 dead links (caused by STP). This is normal
4. If your packet tracer simulation starts to lab OR your links are all green you have caused a broadcast storm by reconfiguring the etherchannel (most likely an active/passive issue) 

![3SwitchEtherchannel](/media/3SwitchEtherchannel.png)

### Spanning tree

The types of spanning tree protocols are listed below. Read the table on X and Y to find the type of STP based on its encapsulation and if its normal/rapid.

For example: Normal STP using ISL is called STP

| Encapsulation | 802.1q | ISL    | both    |
|---------------|--------|--------|---------|
| Normal        | STP    | PVSRP  | PVSTP+  |
| Rapid         | RPSTP  | RPVSTP | RPVSTP+ |

#### Some important notes
* PVSTP and PVSTP+ are the default STP options
* By default any switch on PVSTP should be changed to PVSTP+ for enhanced performance
  * `spanning-tree mode rapid-pvstp`
* BPDU guard or portfast should NEVER be on a trunking port, STP allows for a switch to be vulnerable with these options enabled. Keep it for access ports.

#### Desg Altn Root

These describe what each port on your switch is doing. Have a look at this information through `show spanning-tree`
* Desg [fwd] - Away from root switch
* Altn [blk] - Backup route
* Root [fwd] - Towards root

#### Force a switch to become root

```
spanning-tree root primary
! OR
spanning-tree root secondary

! Rig the STP election to influence a switch into becoming root
spanning-tree vlan 1,2,3... priority [priority]
```

#### Change a ports cost

```
S1(conf-int)#spanning-tree cost [cost]

! You need to include the cost to undo this command
no spanning-tree cost [cost]
```

#### Force a switch to be root on many VLANs

```
spanning-tree vlan 1,2,3... root primary

! or make it secondary
spanning-tree vlan 1,2,3... root secondary
```

### BPDU guard

BPDU guard is a small topic in N&S

A BPDU guard is a CISCO feature that allows you to defend against a BPDU attack which exploits a vulnerable switch with another malicious switch negotiating new paths or routes etc.

A BPDU guard should only be places on an **access port**. If you place it on a trunking port it will break your network.

```
int f0/1
spanning-tree bpduguard enable
```

### Portfast

Portfast is a small topic in N&S

Portfast simply allows an **access port** to go into a forwarding state (see STP) faster. If you place it on a trunking port it will break your network.

```
int f0/1
spanning-tree portfast
```

## Preparing for assessments

This should account for around 15% of your knowledge/effort put into the unit.

This guide isnt really an be all end all solution and doesn't contain everything you need. As bad as the lectures are you should still watch them. But more importantly practice the labs. They will not only make your practical skills better but also prompt you to discover the answers to the exam questions as you solve your own practical problems through your google degree.

The only bit of advice i have is to be aware of all the *'show'* commands. I have abbreviated them as much as possible for speed during the exam.
```
! Show interfaces
sh ip int br

! show vlans
sh vlan br

! show all the sub interfaces on a router
sh ip route

!show the trunking interfaces
sh int trunk

! show the switchport security report
! this is useful to see the learns macs and their associated ports
sh port-security
```

Another useful technique to make use of is performing a grep command on your running configuration. Cisco machines after all are compatible with some POSIX standards (heavy asterisk).

```
! Eg. I want to see what type of STP i am running
sh running-config | include spanning-tree mode

! Theres no need to include quotes in your query.
! Though make sure to include a space around the pipe: 'command | search'
```

### Exam run through

I plan on recording a live run through of the exams as its easier to practice by following with me as i do the exam, rather than reading notes

#### Midterm exam

#### Practical exam

### Gotchas

#### Jason gives you a L3 switch for your exam
sometimes Jason will provision a L3 switch in your exam when most students are used to L2 switches in their labs. You will know if this is the case (given the hardware hasn't changed) in ATCs labs because your switches interfaces will be 3 numbers long, for example `f0/0/0`.

dont panic! Three numbers just means your switch has more than the 24 ports on the 2960 that ATCs labs currently have. `f0/0/0` works in the same way to `f0/0`.

#### You are missing a MOTD or Hostname

#### You forgot your loopback

Both skills exams **WILL** ask you for a loopback address. Your lab/exam network will work without it so students may not catch it as an error. On the midterm exam you will fail the exam. And you may fail the final exam or have marks taken off if you miss it.

#### Router on a stick, ROS, sub-interface routing, intervlan routing

*Router on a stick* (ROS) and *sub interface routing* and *intervlan routing* are **the same thing**. The practical exams may use these interchangeably.
These terms are just referring to the g0/0/1.x (sub interface) of a router.