
# Week 1 Lecture 1 - Security Concepts

## Describing a security system.
* Vulnerability. A weakness in the system. Theres something in this system that could be exploited
* Risk. A possible event that could exploit a Vulnerability
* Threat. a method for triggering a risk
* Countermeasure. A way to stop the threat from triggering a risk
* Assurance. the level of guarantee that a system will behave as expected

## Security Vs Functionality Vs Cost
>Security always involves balancing tradeoffs

Security involves tradeoffs (Security / Functionality / Cost). There is no perfect security system that achieves all 3 pillars of security.

## Proportionate security
The level of security should be proportionate to the value of the information you are trying to protect.
* You need to decide if the cost of this security measure is justified for the information that we need to protect. 

# History of network security
## Hacking in the 70s
> The late 70s and early 80s saw large scale commercial networks deployed. These networks were secured via physical isolation.

### Phone Phreaking  
Most hacking that ocurred in this time was in telecommunications
* unauthorized use of the phone system to make expensive long distance phone calls
* Attacks on metering, signaling, and switching systems

## Hacking in the 80s
Corporate networks started moving to X.25 networks which were more secure. ATMs also started to be rolled out.
* X.25 is a predecessor to the frame relay

## Hacking in the 90s
* The internet is born!
* Transaction systems are still based on X.25
* The internet has no real security measures in place
* Encryption development starts in the late 90s

## Hacking in the early 2000s
* The internet *Boom* happens. commercialization starts to happen
* work on encryption really starts. public key infrastructure allows internet transactions feasible for the first time
* By the mid to late 2000s there was a lot of hacking tools and exploits discovered and made popular in the general public
  * Hacking scripts
  * WLAN (wireless lan) exploits

## Hacking today
* Death of privacy
* Cloud computing
* Smart phones
* Social networks
* VoIP
* 3G, 4G, 5G

# Writing security policy (introduction)
> Things to consider when writing a security plan

## Security policy considerations
The main considerations to make when writing a security plan are...
* Manage the tradeoff between Security / Functionality / Cost
* Work out what risks are acceptable and what are unacceptable
  * *"It is not possible to be connected to the internet and have no risk."*
* A security program needs to be implemented in a top down manner. see 'Describing a security system'
* Your policy needs to be understood and reviewed by executives and upper management. It needs to be readable


## Confidentiality and Integrity
Confidentiality is the secretiveness of information. Integrity is the security of the information. 

> Useless information kept in fort knox has **high** integrity but **low** confidentiality. 

> Whereas company secrets stored on a public website has **high** confidentiality but **low** integrity

#### Confidentiality
* What information needs to be kept secret? How secret does it need to be?
* What level of confidentiality should be used for different information. Eg credit card information Vs customer addresses Vs stock lists.

#### Integrity
* How timely, accurate, and integral should information be?
* what is the importance of different information?
  * How careful are we to not lose that information?
* What is the integrity of different types of information
  * Transactions should have high integrity etc
* How is information going to be secured?
  * Digital signing, physical isolation, passwords?

### Availability
How available is information. Generally the more available information is the more expensive it is
* How available is it to certain people?
* Can an employee access confidential information?
* Where is information available physically?
  * Cardboard box? Secure server? Offsite in a nuclear bunker?

# Appendices
## Frame Relay Vs PTP network
* A Point To Point network connects 2 Routers to each other DIRECTLY
* A frame relay is a packet switching technology that can use virtual circuits to communicate via a Frame relay switch that is at the ISP. packets are switched using a DLCI.
![diagram](https://ccieblog.co.uk/wp-content/uploads/2012/10/OSPF-Non-Broadcast-Network.png)
