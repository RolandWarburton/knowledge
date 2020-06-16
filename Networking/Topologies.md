# Topologies

A topology is how a network connects and communicates with different devices.

### Star Topology

![starTopology](/media/starTopology.png)

* Most common topology
* All clients connect to a central point. Eg. a hub or switch
* All data must pass through the central point before continuing to it's destination

<!-- table -->
| Pros                                                                     | Cons                                                                                             |
| ------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------ |
| If a computer or cable fails. The other computers  will not be affected. | A single point of failure network. <br>ie. If the central hub or switch fails, everything fails. |

### Bus Topology

![busTopology](/media/busTopology.png)

* Old technology not used today
* Each computer is connected to a single backbone ([coaxial cable](https://en.wikipedia.org/wiki/Coaxial_cable))
* Computers connect to the coaxial cable with a connected called BNC connectors (or T connecter for short)
* Used in the [10BASE2](https://en.wikipedia.org/wiki/10BASE2) and [10BASE5](https://en.wikipedia.org/wiki/10BASE5) standard which are part of the [IEEE 802.3a](https://en.wikipedia.org/wiki/IEEE_802.3) standard.
* Must terminate on both ends

<!-- table -->
| Pros                        | Cons                                                                                                                                          |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| Cheap and easy to implement | <br>Cables must be terminated on both ends with terminators. <br>If there is an open end, signal reflection will occur (disrupted data flow.) |

### Ring Topology

![ringTopology](/media/ringTopology.png)

* Each computer is connected to each other in the shape of a ring (closed loop)
* Each computer has at least two neighbors
* A packet is sent around the ring until it finds a neighbor
* Rarely used today

<!-- table -->
| Pros                             | Cons                                                                                                        |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| Easy to install and troubleshoot | <br>If one computer goes down or if there is a break in the cable. <br><br>All data flow will be disrupted. |

### Mesh Topology

![meshTopology](/media/meshTopology.png)

* Little chance of total network failure
* Mainly used on WANS
* Rarely used on LANS

<!-- table -->
| Pros                                    | Cons                                          |
| --------------------------------------- | --------------------------------------------- |
| Little chance of whole network failure. | Expensive and creates a high redundancy level |

### Hybrid Topologies

![hybridTopologyA](/media/hybridTopologyA.png)

![hybridTopologyB](/media/hybridTopologyB.png)

* Best of both worlds
* Common hybrids include
  * Star bus (hybridTopologyA)
  * Star ring (hybridTopologyB)

### Point to Point Topology

![point2pointTopology](/media/point2pointTopology.png)

* 2 network devices connect to each other over a single cable
* Can be anything. Eg. computers, switches routers, servers
* Simplest topology

### Peer to Peer Topology

![peer2peerTopology](/media/peer2peerTopology.png)

* Similar to Point to Point but with multiple computers
* All the clients on a network can talk to each other and share resources
  * Eg. a computer can share it's printer with the other computers on the network
* No centralized server to connect to
Typically found in homes and small businesses

### Client to Server Topology

![clientToServerTopology](/media/clientToServerTopology.png)

* Clients connect **directly** to a central dedicated server rather than connecting to each other
* Storage efficient, you don't need to store resources on each computer

### Point to Multipoint Topology

![point2multipointTopology](/media/point2multipointTopology.png)

* A client connects to a  central wireless base station (access point)
* Similar to a star network in that clients cannot directly communicate to each other
