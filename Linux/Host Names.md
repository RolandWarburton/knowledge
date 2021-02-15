# Host names and Domains

```output
The name of the user
  |
[user@hostname ~]#
         |
       name of the computer
```

## Hosts

### /etc/hostname

Information about **THIS** computer.

### /etc/hosts

Information about **OTHER** computers, with the exception of the default 127.0.0.1 which is a loopback (equivalent to localhost).

In fact the whole reason that *localhost* works in the first place is because there is a mapping between 127.0.0.1 and localhost. This mapping can be seen in /etc/hosts as follows.

```output
  This address
    |
127.0.0.1	localhost localhost.localdomain
               |
             Can be known as any of these things
```

So now when you go to *"localhost"* on your computer, it is resolved to 127.0.0.1 using the hosts file which is the actual host (IP) that the computer understands (basically DNS but just locally).

### Domains

The concept of "DNS but locally" is known as a **domain** and its purpose is to unify a network of computers under a single name.

An example of this would look as follows.

```output
The name of the user on the domain
  |
roland@mynetwork.domain
           |
         Some domain name. Eg lab.lan or business.com
```

### Config example

Lets say you are setting up a brand new server, lets say the function of this server is to serve webpages that allow employees to login to a sales system (just as an example).
So we have 2 objectives.

1. We want to easily identify the server as a webserver so lets give it a hostname of **"webserver"**.
2. We want employees to easily access the server without using its IP so lets give it an easily identifiable URL of "webserver_login.com"

#### Step 1

On the webserver: Lets call this server "webserver".

```none
sudo vim /etc/hostname
```

```output
# /etc/hostname
webserver
```

#### Step 2

On the webserver: Lets modify `/etc/hosts` and add an entry for "webserver_login.com" for the server itself. For this we can use its routable IP (like 192.168.0.1 or 10.0.0.1 etc), however a better solution is to use its loopback address 127.0.0.1.

```none
sudo vim /etc/hosts
```

```output
# /etc/hosts
#                                         Add as many different names as you want
#                                           |
127.0.0.1 localhost localhost.localdomain webserver_login.com
```

However now we have an issue! How does the employee/clients computer know how to route to webserver_login.com? The answer is that it doesn't (not without telling it how to first).

Lets solve this my manually adding webserver_login.com to the employees computer in the same way that we associated *127.0.0.1* with *webserver_login.com* on the webserver. In this example lets say the rotatable IP of the webserver is 10.0.0.1

On the clients computer.

```output
# /etc/hosts
#
# Associate this IP
#  |
10.0.0.1 webserver_login.com
#           |
#         With this domain name
```

Done! Now when the client tries to access webserver_login.com it will be resolved to 10.0.0.1 using /etc/hosts.

However there is a better way of automating this distribution of known hosts using a **domain name server** which i will cover next.

## Domains - Setting up a domain name server

Domain name servers provide a source of truth for clients on a network to automatically handle the association of IP to Domain names that clients require to use use network names.

To set up a domain name server you can use the following steps.

In the example the IP address of the DNS server is **10.10.10.10**. We will use a client on **192.168.0.100**. The name of the domain will be **lab.lan**, and the name of the DNS server will be **auth**. With that done lets get started!

First verify your hostname of your DNS server

```none
hostnamectl
```

```output
Static hostname: auth.lab
		Icon name: computer-vm
		Chassis: vm
	Machine ID: e20be31b1c144cd6a0ebbd720dd7018e
		Boot ID: 0c53d3d2e7894439b7f3d52f3adb0613
Virtualization: vmware
Operating System: CentOS Linux 8 (Core)
	CPE OS Name: cpe:/o:centos:centos:8
		Kernel: Linux 4.18.0-193.14.2.el8_2.x86_64
	Architecture: x86-64
```

Then install packages

```none
dnf install bind bind-utils
```

Then edit the (idk what this is)

```none
vim /etc/named.conf
```

```none
# line 11: Add your static IP to the listen-on port
	listen-on port 53 { 127.0.0.1; 10.10.10.10; };

#line 19: allow any query
	allow-query {localhost; any;};
```

Start the domain name service

```none
systemctl start named && \
systemctl enable named && \
systemctl status named
```

Poke holes in the firewall for DNS ports so they wont be blocked when a client makes a request.

```none
firewall-cmd --zone=public --add-service=dns --permanent

# OR
firewall-cmd --permanent --add-port=53/tcp
firewall-cmd --permanent --add-port=53/udp

# THEN RUN
firewall-cmd --reload
```

Make sure the firewall is allowing ports 53/tcp and 53/udp using `firewall-cmd --list-all`.

```output
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens192
  sources:
  services: cockpit dhcpv6-client ssh
  ports: 53/tcp 53/udp
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

Next lets make a DNS zone within `/etc/named.conf`. At the bottom of the file define a zone.

```none
zone "lab.lan" IN {
	type master;
	file forward.lab.lan;
	allow-update { none; };
};

# reverse 10.10.10.1(0) and drop the last char (in brackets)
zone "1.10.10.in.addr.arpa" {
	type master;
	file "reverse.lab.lan";
	allow-update { none; };
};
```

Then create the forward files we just referenced, we will use this file to base our reverse file after.

```none
cp /var/named/named.localhost /var/named/forward.lab.lan
```

Then edit /var/named/forward.lab.lan (make sure to include '.' delimiters).

```none
vim /var/named/forward.lab.lan
```

Change the 2nd line to use your domain name. DNS will resolve "host" and "auth" to 10.10.10.10

```none
$TTL 1D
@	IN SOA auth.lab.lan. root.lab.lan. (
				0		; serial
				1D		; refresh
				3H )	; minimum

@		IN		NS		auth.lab.lan.
@		IN		A		10.10.10.10
auth	IN		A		10.10.10.10
host	IN		A		10.10.10.10
roland	IN		A		192.168.0.100
```

Now create the reverse file

```none
cp /var/named/forward.lab.lan /var/named/reverse.lab.lan
```

```none
vim /var/named/reverse.lab.lan
```

"10" is taken from the 10 in "10.10.10.**(10)**" to reverse lookup auth.lab.lan

"100" is taken from 192.168.0.**(100)** to reverse lookup roland.lab.lan

```none
$TTL 1D
@	IN SOA auth.lab.lan. root.lab.lan. (
				0		; serial
				1D		; refresh
				3H )	; minimum

@		IN		NS		auth.lab.lan.
@		IN		PTR		lab.lan.
auth	IN		A		10.10.10.10
host	IN		A		10.10.10.10
10		IN		PTR		auth.lab.lan.
100		IN		PTR		roland.lab.lan.
```

Now fix the file perms so that "named:named" owns the file.

```none
sudo chown named:named /var/named/forward.lab.lan
sudo chown named:named /var/named/reverse.lab.lan
```

Next check that the config syntax passes.

```none
named-checkconf -z /etc/named.conf
```

A successful load will return

```output
zone lab.lan/IN: loaded serial 0
zone 1.10.10.in-addr.arpa/IN: loaded serial 0
zone localhost.localdomain/IN: loaded serial 0
zone localhost/IN: loaded serial 0
zone 1.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.ip6.arpa/IN: loaded serial 0
zone 1.0.0.127.in-addr.arpa/IN: loaded serial 0
zone 0.in-addr.arpa/IN: loaded serial 0
```

Next check the zone files

```none
named-checkzone forward /var/named/forward.lab.lan
named-checkzone forward /var/named/reverse.lab.lan
```

Both the forward and reverse zone should return

```output
zone (forward OR reverse)/IN: loaded serial 0
OK
```

Now go ahead and restart the service

```none
systemctl restart named
systemctl status named
```

Lastly add your DNS server to your routers list of DNS servers. And change your DHCP servers settings to hand out your DNS servers IP as well.

### Testing the DNS server

Now lets go to the client computer 192.168.0.100 and run `hostnamectl`. Observing the following output or something similar.

```output
Static hostname: roland
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 05492ea8bf744c9282896cd5250e5801
           Boot ID: d7aa08c2b2594d5f9a20162ff553523e
    Virtualization: vmware
  Operating System: Debian GNU/Linux 10 (buster)
            Kernel: Linux 4.19.0-9-amd64
      Architecture: x86-64
```

Next lets observe `/etc/resolv.conf`.

```output
# Generated by NetworkManager
search lab.lan
nameserver 10.10.10.10
```

Here we can see that our client knows to use 10.10.10.10 as a nameserver to resolve DNS queries with. The **search lab.lan** derived from my router (PFSense) because the DNS service on my PFSense is also running DNS services (as a DNS resolver for public DNS on 1.1.1.1 and 8.8.8.8).

### Additional debugging

If you dont get the results above. Here are some tips for resolving any issues.

#### Issue: nameserver is 10.10.10.1 (incorrect nameserver)

By default most routers like to give out the DNS (aka nameserver) as itself. In this case the default gateway of the 10.10.10.0 subnet is 10.10.10.1 so thats what a DHCP provisioned device will get. To fix this you should change the DHCP settings on your DHCP server to hand out the correct IP

#### Issue: public DNS queries stop working

This is normally caused by Not enough DNS servers or no BIND for fallback nameservers on named. If 10.10.10.10 is your only source of DNS then a client has no way of being able to reach public stuff (like google).

To solve this you can either.

1. Add another bind in /etc/named.conf. `listen-on port 53 { 127.0.0.1; 10.10.10.10; 1.1.1.1; };` Where 1.1.1.1 is a DNS server that named can look at for DNS it does not have locally.
2. Add an additional nameserver to your router. In this case i am using PFSense, so under *System -> general setup* I can have DNS servers. 10.10.10.10 and 1.1.1.1
