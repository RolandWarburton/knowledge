# Network Components
Stuff that you put on a network.

## Routers and Modems

![modem_and_router](/media/modem_and_router.png)

> A modem connects you to your ISP. A router has **all** the functionality of the router but with switching capability for the internal network too.

* A modem has **few** ports. to connect the outside (ISP) to a router/switch (internal)
* A modem connects to the ISP from very edge of the network.
* A modem handles the external IP


* A router is a modem and a switch combined into one
* A router has **many** ports (around 5)
* A router handles all the responsibilities of a modem (connecting to the ISP) but adds switching capability

## Other Networking Equipment

### DOCSIS Modem

* Data over cable service interface specifications
* Home broadband cable modem
* Handles both incoming and outgoing data signals
  * Internet
  * Video
  * voice
* Supports up to 10G down and 1G up

### Hub

* Has multiple ports that accepts ethernet connections from networked devices
* A hub is a dumb switch in that it has no intelligence as to where data is supposed to go. When a data packet arrives at a port it is copied and sent to all other ports
* There are two different types of hubs
  * passive - requires power
  * Active - requires power

### Switch

* A smart hub
* A switch learns the physical addresses of the devices connected which means it only sends 1 packet to the intended receiver by building an ARP table (among other things)
* Most switches operate at level 2 of the OSI model, however this isn't always the case
  * Multilayer switch - This switch can operate at layer 2 and layer 3 
  * Content switch - This switch can operate at levels 4 through 7. Its expensive and can do advanced filtering and load balancing

### Bridges

A bridge is used to separate networks that have hubs on either side to prevent congestion.

![bridge](/media/bridge.png)

### Router

* Routes or forwards data from one network to another
* An example of this is the router in your house. In addition do being a modem, the router separates your internal network from the WWW
* Routing is performed based on IPs (internal/external/subnets)

### Gateway

A device that joins 2 networks together when a different protocol (**NOT a different medium**) is used on each side

### CSU/DSU
A Channel Service Unit / Data Service Unit is a single piece of hardware that translates between LAN to WAN.
* It needs to exist to ensure that the WAN link between LANs is clocked the same
* A CSU/DSU exists on either side of the WAN

### WAP

Literally a wireless hub

![WAP](/media/WAP.png)