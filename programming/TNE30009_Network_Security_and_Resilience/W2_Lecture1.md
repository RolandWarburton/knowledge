# Week 2 Lecture 1
An introduction and explanation of rainbow tables
1. Taxonomy of attacks and methodology
2. Notable exploits in the past 10 years using rainbow tables
3. Basic attacks using rainbow tables

## Basic concepts of attack taxonomy
* Vulnerability. a software or hardware weakness that can be exploited
* Threat. the potential danger to a resource
* Threat agent. The actor that can exploit that weakness
* Risk. The probability that a threat agent will exploit that risk
* Exposure. An instance of risk/loss from a threat
* Countermeasures and controls. A mechanism to eliminate or limit vulnerability

![Taxonomy of attacks](https://i.imgur.com/DWWk5nD.png)

# Vulnerability types
* Design errors
* Protocol weaknesses
* Software weaknesses
* Misconfiguration
* Hostile code
* Human factors

# Adversaries / Threat agents
* Foreign states
* Terrorists
* Criminals
* Hackers
* Corporate competitors
* Government agencies

# Classic attack structures
An attack on a network may follow a sequence
1. Reconnaissance
2. Identification of operating systems and applications
3. Access (via social engineering)
4. Escalate privileges
5. Gathering additional passwords and secrets
6. Installing backdoors or other exploits
7. Exploiting compromised hosts

# Building blocks of attacks
Most attacks are used to open up more attack options

## Buffer overflow
When the call stack is abused by overflowing addressable space by passing a function too much information so that it overwrites the return to give you access to some part of the code that was not meant to be returned

## Malware
Malicious code that is propogated through the internet. Can take on characteristics as worms, trojans, or viruses
### Virus
Cannot reproduce on its own. needs a vector to attack a new host, such as USBs. Viruses can erase files, encrypt files, send emails, etc...

Virus signatures can be used to identify a virus. The virus will have a unique string of bits that identify it (like a fingerprint)

### Worms
More benign than a virus. Self replicates itself to as many hosts as possible and hogs resources. Modern malware can blend worms and viruses together to make them more harmful

### Trojans
Trojan horses trick the user by piggybacking on software that the user wants to use. Some sophisticated trojans will try and take over the entire machine. Trojans can do most things that a virus can, though trojans are most known for setting up botnets

## Backdoors
A backdoor is a method of accessing a computers resources without needing authentication. Some backdoors are 

## SQL injection
SQL injection and XSS (cross site scripting) take advantage of weak web security to exploit a websites database of attack other users of the website

## Password attacks
> If its easy to remember than its probably easy to guess

Passwords are the most common authentication technique but can be a weak way of proving identity (see the triangle of authentication). Humans are lazy and will tend to pick the easy to remember option, rather than the secure one. Even if security admins try and implement policy that requires passwords to be changed often, it usually creates more headaches than benefits in security.

# Password cracking
Passwords should not be stored in plain text.
* Passwords use hashing to obscure the password with math!
* A hash function takes some digital object (string of text or something similar). The same string will produce the same output
* A hash function cannot be reversed. The hash mush be one way
* A hash function will produce the same length of hash no matter what it is hashing
* When a user authenticates to a system. their password is hashed and then compared to a database of hashed passwords

## Types of hashing
* SHA-1 - produces 160 bit hashes
* SHA-256 - produces 256 bit hashes

# Rainbow tables
Rainbow tables enable a hacker to lookup a hash value to find the corresponding plain text password in a precomputed table. A rainbow table is built ahead of time and takes a **LONG** time to determine every possible hash.
* Tables can be very large (multi TB without reduction functions)
* A rainbow table trades storage space for time. You need a lot of space for the pre-computations. but you gain time when you need to use the table
* Rainbow tables work best on passwords that are hashed with SHA-1 or SHA-256, and Md5
* A rainbow table has one hash type that it can reverse password hashes for. if the table was build for Md5. then it can only be used for passwords that were hashed with Md5
* Rainbow tables can include a salt for hashes. if the the salt is solved ahead of time it increases the chance of recovering a hashed password

> A rainbow table is the process of compressing the hash table down into something more manageable

## Reduction functions
Rainbow tables take up a lot of space. Reduction functions help make them traversable.

p = A set of all letters with a set length (Eg 3)
h = A set of all hash values of p
H = A set of all hashes of any length in the entire rainbow table

```H : p -> h```

R (reverse) maps h to p
R is **Not** a reverse function. It just maps h to p through brute force. Ie it maps h back into the password space.

```R : h -> p```

The output of H is the input of R. But the output of R is not the same as H. So you only need R to map from the hash value (of length 3) to H.

Example
H(abc) = 123456
R(123456) = abc

Given any of these elements in the chain. You can find any elements after it using H and R functions.

## Traversing Rainbow tables

H is used to hash abc. then a unique R function is used for each node in the chain to provide the next node. this is why abc does not turn back into abc, because R1 was used and not abc's specific R function. 

Then H is used again on 'def' which produces another hash. and then R2 is used again to convert it into hij and so on.
```
----H-------R1------H-------R2-----
abc --> 111 --> def --> 122 --> hij
```
The rainbow table will store the Head of the trail so given Given a hash value in the chain. you can find R by just following the chain.

At this stage there is no compression. This is just a way of traversing the rainbow table. This is also where the name for the rainbow table is derived. The many different reductions create a 'rainbow' of different hashes.

## Compressing rainbow tables
Because the Reduction function and Hash function are both built into the rainbow table. You dont need all the middle points in the list. Rebuilding the list as you need it

Example:
```
----R1------H------R2-------H-----
aba --> 231 --> baa --> 321 --> bcc
```

The head is aba. aba is reduced and hashed until it reaches the tail of bcc.
Eventually you will find the hash you were looking for before you reach the tail, if not you wrap around and start from the head, this is why we keep track of **Both** the head and tail.

* This trades some space complexity for computation complexity at a very desirable rate. a table can be compressed from around 3TB down to about 32GB.

### Avoiding duplication in rainbow tables
A problem can occur in a rainbow table where if you are using the same reduction function some duplication can occur in your string of nodes.

To avoid this you can use different reduction functions for each node.

### An example of a rainbow table lookup process
Example:
You are given 114 as a hash value. you need to find the password using the steps a rainbow table would use.

This is the rainbow table provided with the example:
```
Hash----Red-----Hash----Red-----Hash----Red-----Hash----Red-----Hash
411 --> abb --> 141 --> bab --> 114 --> bba --> 144 --> bbb --> 444
-------------------------------------------------^start here--------
```

Note that If you are looking a rainbow table visually you can just look at the value before it and that is the answer in most cases 

1. You feed that into the hash table. The hash table reduces it to bba. This ISNT the password
2. The rainbow table hashes bba and gives you 144
3. The rainbow table then reduces 144 and returns bbb
4. The rainbow table then hashes bbb and returns 444. This tells us that the password we are looking for is in this row
5. Go back to the beginning of the row and reduce the hash 411 to the password/reduction of abb
6. the password/reduction of abb is then hashed to the hash of 141.
7. the hash 141 is then reduced to the password/reduction of bab.
8. the password/reduction of bab is then hashed to the hash of 114. This is the hash of the hash value that we were given in the example question. So that means that the hash of 114 has a password/reduction of bab

### Protecting against rainbow table attacks
The main defence against a rainbow table is to use a salt.
A salt is additional text that you append to a salt to increase the size of the hash which makes decoding it use magnitudes more computational effort.
* A salt is always randomly generated
* A salt may be added onto the end of a hash
* The salt doesn't have to be a secret but it should be for extra security

This ensures that the attacker has to decrypt the individual salt for each user one at a time rather than being able to look up the hash in a database. This defends against rainbow table attacks 

# Social engineering
Social engineering manipulates humans to gain confidential information.

## Types of attacks
* Phone raffles
* Unsolicited surveys for 'customer satisfaction'
* Phishing attacks on specific people

## Defences for social engineering
* Training employees to report social engineering without any consequences to their employment
* What ways can you detect social engineering attacks on a company?
* Engineering responses in case a social engineering attack takes place
  * Having policy set in place to lock down all confidential information and services in case a password or account is compromised.

# TCP/IP Threats and vulnerabilities
Specific attacks on TCP/IP

> There are many network layer attacks on TCP/IP. such as...
* packet sniffing and password attacks
* IP spoofing
* Sequence number prediction (attacking TCP handshakes)
* TCP Hijacking

## IP spoofing
Change your IP to another IP that might have privilege on the network.
* The attacker makes it appear that a packet was sent by a different machine
* This exploits the trust that 2 machines might already have with each other based on their IPs

### Defense against IP spoofing
* Limit use of trusted machines, 
* Firewalls, ingress filtering (Any internal packets transmitted outside your network with a source) address outside your network should be dropped
* Egress filtering

## TCP sequence hijacking
### TCP sequence number attack
Hijack a TCP handshake to get an authentication token.
You can use a random sequence of numbers to try and defend against this type of hijacking.

### Man in the middle an authenticated session
You can hijack an existing session by performing a man in the middle attack. This also exploits the TCP handshake by trying to intercept an ACK packet to take over the session from another used once they are authenticated in a system.

## DDOS attacks
* Network based DOS attacks (bringing a network down with spam)
* The detection can be difficult. How can you differentiate between legit traffic and an attack?

## SYN flooding
* A type of DOS attack
* server receives LOTS of connection requests and set up resources to deal with the connection request
* If a SYN attack is in place, new connections to the server will be unable to connect until the existing ones have timed out
* A syn attack can be created by spoofing a rotatable but unreachable source address

## ARP poisoning
Corrupts the arp tables of every host in the network so that they contain random garbage
* An IP is mapped to a MAC
* The arp table initially maps the IP to the MAC correctly on both devices
* The idea of arp poisoning is an attacker comes in and somehow sends a message to a host (like lazy arp) that pretends to be an existing device in the ARP table (but has a different IP) so the mapping in the hosts table is updated with the incorrect ARP mapping
* The messages from the host will now go to the attacker because the ARP table has been changed
  * The attacker could also MAP the hosts arp table so that it drops the packet

### Defences against ARP poisoning
* DHCP snooping to check that not too many ARP tables have changed
* Monitor the arp tables to make sure that connections are not being corrupted
* Monitor for changes in important, usually static entries in ARP
* Prevent physical access to the network to prevent an attacker from access to ethernet (can still be attacked over wireless)

## HTTP
* HTTP is a web protocol which has lots of general web exploits
* SOAP exploit
* Social engineering attacks (phishing)
* Attacks where the server is the victim (like DOS)
* Attacks where the client is the victim (XSS cross site scripting)

## DNS
* Maps host names to IP addresses
* Can use DNSSEC to secure the DNS server hierarchy
  * This can prevent spoofing DNS servers (smurf attack)

# Notable TCP/IP exploits
## Conficker
* Can be a worm or a virus
* Consumes resources, disables accounts, block DNS lookups

### Lessons from Conficker
* keep patches up to date
* Strong password!
* Avoid use of trusted hosts
* Control moveable media

## Stuxnet worm
* Known for attacking an uranium refinery in iran
* Targeted the programmable logic controller
* PCL reached the devices through USB and took advantage of unpatched windows software

## Athens wiretapping scandal
* 100s of mobile phones were tapped illegally
* The gvt has to build in wiretapping capabilities
* Hackers gained control over this wiretapping server
