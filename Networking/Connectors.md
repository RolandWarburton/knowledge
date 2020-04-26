# Connectors

### Defining Connector Positions and Contacts

Positions and contacts can be abbreviated to 4P6C (4 positions, 6 contacts).

Usually it only makes sense that a cable standard has the same number of positions as it has contacts. Though sometimes the numbers might be different. 

### Common Jack Identifiers

* "C" - Identifies a surface or flush mounted jack
* "W" - Identifies a wall mounted jack
* "S" - Identifies a single-line jack
* "M" - Identifies a multiple-line jack
* "X" - Identifies a complex multiline or series-type jack

### Ethernet cable types

* RJ-22: handset cable (4p4c)
* RJ-12: basically the same as RJ-11(6p6c)
* RJ-45: ethernet (8p8c)

### Single vs Multimode fiber

#### Multi mode fiber

* Shorter distance than single mode (up to 2km)
* Usually orange in color

#### Single mode fiber

* Longer distance (up to 200km)
* Usually yellow in color

## Data/Ethernet connectors

### RJ-11 (6P4C)

![rj11](/media/rj11.png)

* 4 wire connecter
* "Telephone over ethernet"
* Usually connects to modems from wall plates inside a house.
* Larger than rj-22 (4p4c handset to phone)
* Small version of the rj-45 (8p8c standard ethernet cable)

### RJ-45 (8P8C)

![rj-45_8P8C](/media/rj-45_8P8C.png)

* Standard ethernet cable
* Larger version of the rj-11
* Uses twisted pair cabling
* Attaches network cards (clients) to switches and hubs

### RJ-48c

![rj-45c](/media/rj-45c.png)

* Similar to RJ-45 with the difference being that the cable uses shielded twisted cabling
* Used with T1 lines (1.544 Mbit/s)
* Wired differently to the RJ-45


### UTP (Unshielded Twisted Pair) Coupler

![UTP_Coupler](/media/UTP_Coupler.png)

Connects UTP cables that have RJ-45 connectors to each other. (internet extension cable)

### BNC Connecter

![BNC](/media/BNC.png)

* Common type of [RF](https://en.wikipedia.org/wiki/RF_connector) connector
* BNC is short for: Bayonet Neill-Concelman
* Supports analogue and digital transmissions of both audio and video
* Common on old bus topologies

### BNC Coupler

![BNC_Coupler](/media/BNC_Coupler.png)

Literally an extender for coaxial cables

### Fiber Coupler

![fiber_Coupler](/media/fiber_Coupler.png)

* Extender for Fiber cables on the same type
* Not to be confused with a fiber adaptor which is for connecting two different cables

### F Type Coaxial

![Ftype](/media/Ftype.png)

* A coaxial cables that is typically used to connect to cable modems
* This is the cable I am using to connect to my ISP right now

### IEEE 1934 (aka: Firewire)

![](/media/IEEE_1394_firewire.png)

* Commonly used to attach some types of peripherals such as: cameras, printers and network connections
* Carries moderate power over the cable to charge cameras etc
* Can carry network data but not really used for it

## Fiber Optic cable types

### MT-RJ

![MT-RJ](/media/MT-RJ.png)

* Small form factor
* MR-RJ Stands for mechanical transfer registered jack

### ST

![ST](/media/ST.png)

* ST stands for straight tip
* Common use in single mode fiber optic cable (orange color fiber cable, like at TC)

### LC

![LC](/media/LC.png)

* Short for "Local Connector"
* Is fiber optic
* Similar jack to the RJ-45
* Commonly used between floors on a building

### SC

![SC](/media/SC.png)

* Also known as the "standard connector"
* Commonly used between floors in a building

## Serial Cable Types

### RS-232 (serial)

![RS-323](/media/RS-323.png)

* Standard cable for serial
* Has variations of the D connector such as:
  * DB-9
  * DB-25


## UPC vs APC fiber cable

UPC and APC are types of connectors for connecting a **fiber cable** to another **fiber cable**.

A UPC or APC couples the cables together in different ways to comply with different tolerances in data loss.

### Example 1
**2 ST fiber cables equipped with UPC connectors:** As you make the connection between the two UPC connectors, their surfaces will make contact, however some light will bounce back due to imperfections in the surface (as seen below.) This creates signal loss.

This type of manufacturing Is cheap and effective over short distances.
![UPC2UPC_Fiber](/media/UPC2UPC_Fiber.png)

### Example 2
To reduce the signal loss in UPC, APC coupling was created.

In this diagram **2 ST fiber cables equipped with APC connectors** are coupled with two slanted surfaces which **reduce** signal loss by bouncing the cable back into the wall of the cable.

![APC2APC_Fiber](/media/APC2APC_Fiber.png)
