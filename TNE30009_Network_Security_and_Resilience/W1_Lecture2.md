# Week 1 Lecture 2 - Implementing Security Policy Technologies
>Policy is a set of steps you can take to manage risks in a network to protect a business from threats and risk.

## Developing policy (recap)
To address risks in a network you implement policy by assessing the situation on multiple fronts
* What are the goals of the business and company?
* what are their priorities?
* Whats most important to the company?
* What are the risks and threats
* What is the environment like internally and externally?

# Implementing security policy
There are 3 ways to implement security policy
* **Technological solutions** (the primary interest of this unit)
* Manual procedures
* Transfer the risk (insurance)

## Policy through procedure
Policies are implemented through procedure

> A procedure may be **manual**. For example *checking a criminal record for a new employee*

> A procedure may be **technical**. For example *blocking all telnet sessions on a web server*

## Picking policy applicable for the situation
Not all policy is applicable for all situations.

When implementing security policy you should ensure that the policy you develop is applicable for the business' situation. For example a **technical** procedure where you *use strong firewalls at the network perimeter* to secure the network may not be useful for a business that uses cloud computing to move large amounts of data around would mean that lots of energy and money would have to be spent on a stupid expensive firewall system 

# Technologies used to implement security policy
## Firewalls
Problem: You need to restrict access to information based on which employee is accessing it and where they are accessing it from

Solution: Firewalls allow you to do this. There are many types of firewalls
* To control what traffic can go where you can use an ACL (Access Control List)
* Stateful firewalls
* Dynamic firewall

## Authentication systems
> The triangle of good overall security policy design takes into account *Confidentiality, Integrity, and Availability*

> The triangle of Access Control takes into account "*Something that we know to prove our identity, Something that we have (HID card), and Something that we are (biometric)*"

Strong authentication by definition should include Biometrics **PLUS** an additional item from the triangle

## VPNs
A VPN creates a private communication channel (tunnel) that can be implemented across public networks.

VPNs can overate at layers (2/3). Other types of VPNs can operate at layers 4.

### VPN communication technologies (Layer 2/3)
VPNs use lots of different technologies to secure themselves
* Proxies
* IPSec
* SSL/TLS
* CHAP
* S-HTTP

### VPN tunnelling technologies (Layer 2/3)
To achieve a secure tunnel a VPN uses different tunneling technologies, such as.
* GRE
* PPTP
* L2TP

### VPN authentication technologies (Layer 2/3)
* Kerberos
* RADIUS
* DIAMETER

### VPN encryption technologies (Layer 2/3)
* RSA
* AES
* IKE
* SHA256

## Intrusion detection and prevention system
>Sometimes it is harder to tell when your system is under attack than actually defending.

A tactic to defend against different attacks on a network is to use intrusion detection. An intrusion detection system has 4 key parts.
1. Uses sensors to collect data
2. Monitors the data to report to a human or computer
3. A resolver (human or computer) determines a response
4. The system is then changed to combat the intrusion


## Anomaly detection
There are some established ways to determining when an intrusion is occurring. In the networking world most of this boils down to monitoring the network for unusual traffic

## Cryptography
> Encryption provides confidentiality and integrity.

Many security technologies are built upon cryptography. Such as VPNs, password salting, or other authentication systems

There are two main types of encryption
* Symmetric key (secret key encryption)
  * DES, AES, RC5
* Asymmetric key (a type of public key) encryption
  * RSA, Elliptic Curve

### Public key infrastructure
This technology allows computers to verify each other in a very secure way using cryptography.

Example uses for public keys
* Signing emails to confirm that it was indeed from you
* Signing financial transactions
* Securely logging into servers using a SSH key instead of a password
* Signing code commits on github to verify your identity


## Wireless and IOT
> The approach to securing a wireless network is entirely different to securing a wired network.

Wireless is inherently less secure than a wired connection on the premise that you require a physical port to plug into a wired network, making it physically more secure. There are also WiFi specific vulnerabilities based on the fact that packets are broadcast through the air and not a cable medium.

It is not usually worth spending too much time/money/effort securing a wireless network. rather you should focus on ensuring that not Confidential or Integral data are being transmitted over the wireless network in the first place.

### Cloud technologies
> When you decide to use cloud technologies in your network. be prepared to use a lot of VPNs!

Cloud presents new challenges to securing a network as ut geographically separates the business from its network.

## Conclusion
* Networks were much more open in the past (no security)
* A perfectly secure network is not possible without infinite money
* There is a methodical way of assessing risk and implementing appropriate solutions (see start of this lecture)
* Wireless and cloud introduces new security risks to networks
* A security policy should implement a mix of manual and technology based procedures (see Implementing security policy)

# Network Technologies
> Access networks will be the focus of this unit 
## Classifying networks by type
>There are many ways of classifying network technologies.
* **Topological location:** Core/Dist/Access
* **Medium:**  Wireless/Copper/Fibre
* **Ownership:** Public/Corporate/Home
* **Coverage:** PAN/LAN/MAN/WAN
* **Capacity:** 1000Ethernet/Cable Modem/ADSL2
* **Purpose:** IoT/Voice/Data


## Classifying networks by technology
There are many types of network technologies. Though most attacks occur at the access layer.

### Access layer technologies
* ADSL
* Cable modem
* Wireless LAN
* UMTS
* GPRS
* WiMax
* HSPA
* LTE

### Core layer technologies
* BGP exploits
* Lawful interception

# Access Networks
> Access networks are the 'last mile' of a network. You plug things into them.

There are lots of different access network types, depending on their uses (see Access layer technologies). 

Not all access networks are available at the same time. The technology used for an access network can change based on these factors.
* Coverage of the network
* Reliability of the network
* Implementation costs
* Capacity of the network
* Level of security required

## Physical layer And Data link layer
> The physical and Data Link layer can be used to classify the type of access network you are looking at.

### Physical layer
* The transmission medium that controls how individual bits are encoded onto a medium (wireless, coaxial cable, twisted pair, optic fiber)
* Nits can be encoded as variations in amplitude, phase frequency or baseband
* Uses multiplexing at this layer (TDM, FDM, CDM) 
  * TDM (Time division multiplexing)
    * A channel is split up into slots
    * You have a certain number of slots per frame
    * Each broadcaster has a certain defined slot per the frame
  * FDM (Frequency division multiplexing)
    * A channel is split up based on its spectrum/frequency.
    * EG. A channel of 30Khz is split up between 3 users. user A gets the first 10Khz
  * CDM (Code division multiplexing) 
    * Will cover later
### Data link layer
* Concerned with the transmission of blocks of bits called frames
* Might implement error correction at this layer
* Multiple users accessing a single channel usually requires a MAC (medium access control) mechanism
  * An example of a MAC protocol being used is with the CSMA (carrier sense multiple access) which  listens to or senses network signals on the carrier/medium before transmitting any data. Allowing the transmissions to be controlled.

## Connectionless Vs Connection oriented networks
A connectionless protocol is IP. It doesn't care if the packet has arrived.

A connection oriented protocol is TCP. it DOES care of the packet has arrived. And needs to know that each packet was sent.

## Circuit switched Vs Packet switched
> Packet switched networks are now the norm.

**Circuit switching** is an old technology where electrical currents were manipulated in order to send data down a medium.

**Packet switching** uses IP. A packet has a source and destination address and it does its best to get to its location.


## Security of access networks

### Wireless is less secure that wired
  * Difficult to constrain to specific area (compromises confidentiality)
  * Error prone (compromises integrity)
  * Subject to interference (compromises availability)

### Connectionless networks are generally less secure than switched networks
connectionless networks usually broadcast data which allows sniffing which compromises confidentiality and opens the network up to more compromises.

### Packet switched networks are less secure than Connection oriented networks
Packets can be copied and stored out of a network easier than a stream of random bits can.

# Comparing Specific technologies
## Ethernet
* Frame-based LAN technology
* Defines frame formats, MAC controls (IEEE 802.3)
* uses coax, bridges, hubs, switches
* can be very secure (wired)
  * can be exploited with: mac flooding, arp poisoning

## Cable Modems
* A cable television standard
* uses DOCSIS as a primary standard (Data Over Cable Service Interface Specification)
  * Supports delivery of Ethernet frames between the cable modem and the head-end
  * Specifies a MAC (Medium Access Control) mechanism for sharing of channels
### The DOCSIS Architecture
* DOCSIS uses a Cable Modem (CM) at the Customer premises 
* DOCSIS uses a CMTS (Cable Modem Termination System) at the head-end
* DOCSIS has some useful built-in security mechanisms
* CMTS security provided by ‘hooks’ to other services (such as authentication via 802.1X)
  * DOCSIS security has two protocol components: 
    * an encapsulation protocol for encrypting packet data across the cable network
    * a key management protocol for providing the secure distribution of keying material from the CMTS to client CMs.
* Uses CATV (not cat 5. The V stands for television) infrastructure for data networking
* Usually an asymmetric network with a high bit rate forward link, and a low link in reverse

## ADSL and VDSL
* Uses existing telephone networks to provide broadband
* The first standard to do this was ISDN
* The most important implementation is now xDSL

## xDSL Architecture
### How it works
At each end of the analog loop an xDSL transmission unit modulates the bitstream onto the local analog loop using frequencies above those used by the telco

### xDSL is inherently secure
A super secure Point to Point wired medium. It is a very low level technology and any risks will exist at a higher layer.


## Wireless LANs
* wireless LANs (IEEE 802.11)
* wireless LANs operate in the ISM band. you do not need a permit
* All 802.11 networks use a common MAC layer, but vary in the physical layer
* Common standards of 802.X are 802.11g and 802.11n which make up part of the ISM (industrial, scientific and medical) band
  * The ISM radio bands are portions of the radio spectrum reserved internationally for industrial, scientific and medical purposes other than telecommunications.
* 802.11n uses MIMO (a technique to get more bandwidth out of the same spectrum size) transmitter/receiver design
* 802.11ac is a consumer standard that reaches a theoretical limit of 7 Gbps depending on its configuration

### Security in WLANs
* Historically a pretty weak area 
  * Its wireless
  * Signal reaches a large area
  * Authentication is required otherwise the network can be back door'ed to access restricted parts of the network

### WLAN security standards
* WEP (no longer in use)
* WPA / WPA2 (IEEE 802.11i)

## Long Term Evolution (LTE)
* Sometimes “4G”
* 4G and onwards 
* 4G separates the radio access network and core network
  * LTE is the radio access part of the network that is the access layer component of the network
  * EPS (Evolved Packet System) is the network core component of the network

## 5G NR
* Fifth generation of cellular
* three types
    * eMBBEnhanced Mobile Broadband
    * mMTCMassive Machine Type Communication
    * URLLC Ultra Reliable and Low Latency Communications
* eMBBis a straightforward evolution of LTE
* mMTC (massive machine type communication) will adapt existing cellular network technologies intended for IoT mainly NB-IoT and LTE-M
* URLLC (ultra reliable low latency communication)

Bluetooth (IEEE 802.15.1)
* A cable replacement spec
* Short rance, low bit rates
* Not really an access technology (but can be used with one)
* has plenty of exploits

## IoT network technologies
* Tons of general types: 
  * LoraWAN
  * Bluetooth Low Energy
  * Bluetooth Mesh
  * 6LoWPAN
* personal: 
  * ANT and ANT+
* Home automation: 
  * ZigBee integrated with IP
* Cellular
  * SigFox
    * NB-IoT
    * LTE-M (LTE for Machine to machine communication)
* Industrial Internet of Things
  * PROFINET
  * TSN
  * UA
  * OPC

# TCP/IP

## Topics
* Review of TCP/IP Protocol
* Overview of vulnerabilities
* IPv6 approach to security

## Objectives
* Describe (briefly) the TCP/IP protocol suite
* Describe (in general terms) its vulnerabilities
* Describe (briefly) the IPv6 approach to security

## TCP/IP Protocol:
* IP, TCP
* HTTP, SSH

## TCP/IP security issues:
TCP and ip are old protocols and have had security build into them over time

## IPv6
* Modern version of IP
* Removes the need for NAT (through sheer number of addresses)

# IP
## Connectionless Datagram Delivery
Connectionless
* No predetermined path for transfer of packets
* Each datagram contains a hierarchical destination address
* At each hop, the router decides where the packet is to go

## Datagram
* Data packaged in chunks referred to as datagrams

## Routing IP datagrams
* forms of delivery: direct(destination on network)/indirect(destination not on network)
* relies on routing and ARP tables
  * security issues from this: arp tables can be exploited (arp poisoning)
  * forged routing attacks
  * mac spoofing
  * these issues effect all packet based networks
* Multicast
  * Each subnet has a number of multicast addresses and a broadcast address
  * Multicast allows all hosts in the multicast group to be communicated with through a single IP address
  * The broadcast address is used to transmit a message to all members of the subnet
  * Can be used for a denial of service attack (smurf)

## ICMP protocols
* Can be exploited to get a map of the network which can help hackers identify open points in a network
* Used for
  * Echo and echo reply (ping)
  * Finding a network destination
  * Source quench
  * Router advertisement and solicitation
  * Subnet mask request and reply

# Analyzing UDP/TCP protocols
## UDP protocols
* Largely a framing mechanism for IP packets
* Has a source and destination port
* Security issues
  * No mechanism for reducing the packet rate
  * Can force out TCP connections through brute force

# TCP
## TCP protocols
> A reliable protocol over an unreliable medium

* A reliable transport mechanism
* Uses full duplex
* A stream orientated (sequence is preserved), Virtual Circuit Connection
* Uses Buffered transfer
* TCP gets reliability through the three way handshake (syn/ack/syn-ack)
* Can use timeouts to identify when a packet is lost
* Uses a sliding window
* Uses **source** and **destination** ports to get packets to their target destination **AND** to know which source generated the packet and which program on the receiving end should receive it
* Can take advantage of full duplex (send data both ways)

### TCP buffered transfer
* Datagrams are received at the destination and their content reconstructed in a buffer for processing
* Allows for reliable transfer
* Missing data can be resent

## TCP three way handshake:
data can only be transmitted after the handshake has been completed
* URG urgent
* ACK acknowledgement field is valid
* PSH push this segment
* RST reset the connection
* SYN synchronies sequence numbers
* FIN finish communication

![three way handshake](https://i.imgur.com/e7xWM2Y.png)

1. A: ISN (or SYN) of 2000 is transmitted.
2. B: Receive SYN and ACK 2001. Send own SYN (6587)
3. A: Receive SYN from B and ACK back (6588)
4. A: Start sending data. (data: 2002, ACK 6588)
5. A: Start sending data. (data: 2003, ACK 6588)
6. A: Start sending data. (data: 2004, ACK 6588)
7. B: Send an ACK back to A to acknowledge that it received packages up until ACK 2004 (ACK 6588)
8. A: ACK that it got B's ACK (data: 2005, ACK 6589)

A certain number of messages can be transmitted without receiving acknowledgement of reception. This is called a sliding window and a **bigger** sliding window is when you transmit more packages without receiving acknowledgement

## TCP security issues
* Spoofing. a host pretends to be another host if ident is based on IP
* TCP sessions can be hijacked once the handshake is established
* SYN flooding 

# Domain name system (DNS)
* Maps domain names to IPs
* uses a hierarchy (local/national/global services)
* Domain names can be spoofed
* DNS can be secured with DNSSEC (using digital certificated)

## DNS security issues:
* Compulsory collection of metadata (asking for data leaks)
  * Governments need to keep metadata for up to 2 years
* End points of clients and servers can change and cause wrong information in domain servers
* DNS servers can be spoofed

# NAT
* Maps internal IP and ports to external addresses and ports
* Comes in 2 flavors (NAT and NAPT)
* Hides internal structure of a network by mapping many internal IPs to 1 or few external IPs
* Can make IPSec hard to implement
* Breaks FTP (but not SFTP i think)

## NAPT
* Network Address Port Translation
* Allows for port number overloading

# DHCP
* Configures devices with IPs automatically
* client asks for an IP (transmits a discover)
  * may get more than 1 response which the host chooses to reply to ONE

## DHCP issues
* Could be a malicious DHCP server
* No default controls on number of IPs requested

## IP addresses as time sensitive tokens
* DHCP, NAT, and DNS can be made temporary through the use of time limited tokens
* IPs given by DHCP can be temporary

### Remote logins
* Telnet/ssh

### Email
Lots of types of Email standards
* SMTP
* POP
* IMAP
* MIME

# WWW (world wide web)
* HTTP protocol
* works on port 80 which makes it a vulnerable port
  * SOAP protocol allows for remote procedure calls through port 80 which expands the use of port 80
  * Port 80 is used for a lot of things that it wasnt intended for (like SOAP)

# realtime applications
* Real time transfer (RTP)

# SNMP
* Simple network management protocol
* Used to monitor control network and links

# IP6
* Inherently more secure
* Better support for IPSec
* ICMPv6 has encryption and authentication is baked in

# Week 1 Revision
* What are the ways of classifying networks?
* Connectionless Vs Connection oriented
* Packet Switched Vs Circuit Switched
  * Up until 3G everything was circuit switched
  * 4G onwards everything is packet switched
  * most things today are packet switched
* What are the inherent security of mediums
  * Wireless (least secure)
  * Optical (most secure)
* Different sorts of networks for different applications
* TCP/IP
  * security features (or lack of security features)
  * TCP is connection oriented (three way handshake exchanging sequence numbers to sync transmission)
  * IP is connectionless (unreliable but faster)
  * Inherent security of HTTP, DNS, SMTP, SMTP, SNMP, DHCP
