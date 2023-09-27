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

In the example the IP address of the DNS server is **10.0.0.10**. We will use a client on **192.168.0.100**. The domain will be called **rolandw.lan**. With that done lets get started!

First verify your hostname of your DNS server

```none
hostnamectl
```

```output
Static hostname: auth
      Icon name: computer-vm
        Chassis: vm
    Machine ID: 407289a50273477da74551659b7881a8
        Boot ID: 4a98bb65433d4987bc327ae18ef57ac3
Virtualization: vmware
Operating System: Debian GNU/Linux 10 (buster)
        Kernel: Linux 4.19.0-14-amd64
  Architecture: x86-64
```

Then install packages to work with dns

```none
apt install -y bind9 bind9utils bind9-doc dnsutils
```

### Making bind into a recursive query server

We want to be able to query our local DNS server for any hostname on the internet, of course our DNS server doesn't know the correct mapping to every hostname, so it must query other DNS servers iteratively.

A recursive query is used against the DNS server because the client needs a guaranteed answer from the dns server to which IP a hostname belongs to, not a referral link, see [this video](https://www.youtube.com/watch?v=PS0UppB3-fg). The flow that our client will follow when resolving a domain will look something like this.

1. Client wants to go to google.com
2. Client asks DNS server what is the IP of google.com recursively
3. The DNS server iteratively queries the root domain hints server and a response with a a referral to the .com domain server
4. The DNS server iteratively queries the .com domain server, and the .com server responds with a referral to googles DNS server
5. The DNS server iteratively queries the google DNS server and gets a response with the IP to google.com
6. The DNS server hands that IP back to the client

The reason why we would make our own DNS server recursive, whereas a public DNS server remains recursive is to reduce the load on the public DNS server doing multiple queries.

When an iterative request is made only **one** total query is processed. The iterative server will not ask any other DNS servers for the IP, instead it replies with a referral IP to a closer DNS server using the DNS hierarchy.

When a recursive request is made **multiple** total queries are processed because the recursive DNS server MUST have a response at the end of the day and will keep querying all its top level domain servers until it exhausts all possible options, this costs more computation however must be done to ensure a resolution of the hostname. This process of using a recursive DNS server also benefits the entire network as the DNS server can cache its results to reduce future lookups.

![dns_flow.png](./media/dns_flow.png)

[source](https://www.youtube.com/watch?v=PS0UppB3-fg)

To implement recursion into bind, edit `/etc/bind/named.conf.options` and add the following in the options block.

```none
// Recursion feature =====================================================
// hide version number from clients for security reasons.
version "not currently available";

// optional - BIND default behavior is recursion
recursion yes;

// provide recursion service to trusted clients only
allow-recursion { 127.0.0.1; 192.168.0.0/24; 10.0.0.0/24; };

// forwarders to query
forwarders { 1.1.1.1; 8.8.8.8; };

// enable the query log
querylog yes;
```

### Starting in IPv4 mode

For performance reasons, if you are not going to use IPv6 then theres no need to enable it when starting bind. Fortunately named comes with the `-4` flag to operating in IPv4 mode only, this came pre-configured in debian 10, however heres the steps i used to enable/disable it.

First find all the related service files.

```none
root@auth:~# find /etc/systemd -name "*bind*"
/etc/systemd/system/bind9.service.wants
/etc/systemd/system/bind9.service.wants/bind9-resolvconf.service
/etc/systemd/system/multi-user.target.wants/bind9.service
```

Then edit the correct one (`/etc/systemd/system/multi-user.target.wants/bind9.service`). Observe the **EnvironmentFile**, navigate to that and see that the options for how named is started are located there.

```output
[Service]
Type=forking
EnvironmentFile=-/etc/default/bind9
ExecStart=/usr/sbin/named $OPTIONS
```

```output
#
# run resolvconf?
RESOLVCONF=no

# startup options for the server
OPTIONS="-u bind -4"
```

### Bind directory structure

Note that this changes based on distribution, in this example i am using Debian 10 and it will be different on other distributions like CentOS and FreeBSD.

Bind configurations exist in `/etc/bind`

| File                     | Description                                                      |
|--------------------------|------------------------------------------------------------------|
| named.conf               | The main config file and imports its modules from /etc/bind      |
| named.conf.local         | Contains configuration for the zones that we want to use locally |
| named.conf.default-zones | Dont need to worry about this file for now                       |

### Configuring - Creating zones

Now begin by creating a forward and reverse zone by placing these following blocks in `/etc/bind/named.conf.local`.

#### Forward zone

> A forward zone is responsible for translating a hostname to an IP

![forward_lookup_zone_picture](https://www.mustbegeek.com/wp-content/uploads/2019/03/Understanding-Forward-and-Reverse-Lookup-Zones-in-DNS-Forward-Lookup.png)

[source](https://www.mustbegeek.com/understanding-forward-and-reverse-lookup-zones-in-dns/)

```none
// Domain name will be rolandw.lan
zone "rolandw.lan" IN {
  //Primary DNS
  type master;

  // Forward lookup file
  file "/etc/bind/zones/forward.rolandw.lan.db";
     
  // Since this is the primary DNS, it should be none.
  allow-update { none; };
};
```

#### Reverse zone

> A reverse zone is responsible for translating a IP to a hostname

![reverse_lookup_zone_picture](https://www.mustbegeek.com/wp-content/uploads/2019/03/Understanding-Forward-and-Reverse-Lookup-Zones-in-DNS-Reverse-Lookup.png)

[source](https://www.mustbegeek.com/understanding-forward-and-reverse-lookup-zones-in-dns/)

```none
// Hosts looking for hostnames in the 10.0.0.0/24 range go to this zone
zone "10.0.0.in-addr.arpa" IN {
  //Reverse lookup name, should match your network in reverse order
  type master; // Primary DNS
  
  //Reverse lookup file
  file "/etc/bind/reverse.rolandw.lan.db";
  
  //Since this is the primary DNS, it should be none.
  allow-update { none; };
};
```

#### Zone ACLs

In addition to providing security on a global scope within `/etc/bind/named.conf.options` by adding the following lines.

```none
// range to allow requests from
allow-query { localhost; 10.0.0.0/24; 192.168.0.0/24; };
```

We can do a similar thing on a per zone basis by using an ACL, see [11.2.2.2](https://docstore.mik.ua/orelly/networking_2ndEd/dns/ch11_02.htm).

> BIND 8 and 9 also allow you to apply an access control list to a particular zone. In this case, just use allow-query as a substatement to the zone statement for the zone you want to protect.

```none
vim /etc/bind/named.conf.local
```

```none
acl "CLIENTS" { 10.0.0.0/24; 192.168.0.0/24; };

zone "rolandw.lan" IN {
  ...
  allow-query { "CLIENTS"; };
}

zone "10.0.0.in-addr.arpa" IN {
  ...
  allow-query { "CLIENTS"; };
}
```

### Start the domain name service

```none
systemctl start named && \
systemctl enable named && \
systemctl status named
```

### Firewall configuration

DNS runs on port 53 on tcp/udp. Poke holes in the firewall for DNS ports so they wont be blocked when a client makes a request.

If you are using firewall-cmd, then use these instructions.

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

### Create zone lookup files

Now we need to create the files we references when creating the zones (a fqdm.db file). We will place these databases in `/etc/bind/zones` so create that directory.

```none
mkdir /etc/bind/zones
```

Then copy over the template files from the base bind directory.

```none
cp /etc/bind/db.local /etc/bind/forward.rolandw.lan.db
cp /etc/bind/db.127 /etc/bind/reverse.rolandw.lan.db
```

Ensure that you copy these files, not create them. This guarantees that the file permissions will be correct.
If you need to correct the file permissions for these zone lookup files, then do so now.

```none
chown root:bind /etc/bind/zones/forward.rolandw.lan.db
chown root:bind /etc/bind/zones/reverse.rolandw.lan.db
```

#### forward zone lookup

```none
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     ns1.rolandw.lan. root.rolandw.lan. (
                              3         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
; Comment out below three lines
;@      IN      NS      localhost.
;@      IN      A       127.0.0.1
;@      IN      AAAA    ::1

;Name Server Information

@       IN      NS      ns1.rolandw.lan.

;IP address of Name Server

ns1     IN      A       10.0.0.10
```

#### reverse zone lookup

```none
;
; BIND reverse data file for local loopback interface
;
$TTL    604800
@       IN      SOA     rolandw.lan. root.rolandw.lan. (
                              3         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
;@      IN      NS      localhost.
;1.0.0  IN      PTR     localhost.

;Name Server Information
@       IN      NS     ns1.rolandw.lan.

;Reverse lookup for Name Server
10      IN      PTR    ns1.rolandw.lan.
```

### Verify configuration

Next check that the config syntax passes.

```none
named-checkconf /etc/bind/named.conf.local
```

A successful load will return nothing and a return code of 0.

Next check the zone files.

```none
named-checkzone rolandw.lan /etc/bind/zones/forward.rolandw.lan.db
named-checkzone 10.0.0.in-addr.arpa /etc/bind/zones/reverse.rolandw.lan.db
```

Now go ahead and restart the service

```none
systemctl restart bind9
systemctl status bind9
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
search rolandw.lan
nameserver 10.0.0.10
```

Here we can see that our client knows to use 10.0.0.10 as a nameserver to resolve DNS queries with. It has also inherited the search domain **rolandw.lan** which is derived from the routers domain (defined under "System/General Setup" in PFSense).

## Final DNS Configurations

```output
// /etc/bind/named.conf.local
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization

// I un-commented this - Roland
// https://serverfault.com/questions/306104/what-is-the-point-of-the-zones-rfc1918-file-for-bind9
// It is generally considered a good practice to serve localhost, 0.0.127.in-addr.arpa and the
// RFC-1918 reverse zones on your internal DNS system to prevent sending queries from them out to
// the internet.
include "/etc/bind/zones.rfc1918";

acl "CLIENTS" { 10.0.0.0/24; 192.168.0.0/24; };

zone "rolandw.lan" IN { //Domain name
     type master; //Primary DNS
     file "/etc/bind/zones/forward.rolandw.lan.db"; //Forward lookup file
     allow-update { none; }; // Since this is the primary DNS, it should be none.
     allow-query { "CLIENTS"; };
};

// 10.0.0.0/24
zone "10.0.0.in-addr.arpa" IN { //Reverse lookup name, should match your network in reverse order
     type master; // Primary DNS
     file "/etc/bind/reverse.rolandw.lan.db"; //Reverse lookup file
     allow-update { none; }; //Since this is the primary DNS, it should be none.
     allow-query { "CLIENTS"; };
};
```

```output
// /etc/bind/named.conf.options
options {
        directory "/var/cache/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0's placeholder.

        // forwarders {
        //      0.0.0.0;
        // };

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        dnssec-validation auto;

        // Change from "none" to "any" to listen on ip6
        listen-on-v6 { none; };

        // range to allow requests from
        allow-query { localhost; 10.0.0.0/24; 192.168.0.0/24; };

        // Recursion feature =====================================================
        // hide version number from clients for security reasons.
        version "not currently available";

        // optional - BIND default behavior is recursion
        recursion yes;

        // provide recursion service to trusted clients only
        allow-recursion { 127.0.0.1; 192.168.0.0/24; 10.0.0.0/24; };

        // forwarders to query
        forwarders { 1.1.1.1; 8.8.8.8; };

        // enable the query log
        querylog yes;
};
```

```output
// /etc/bind/zones/forward.rolandw.lan.db
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     ns1.rolandw.lan. root.rolandw.lan. (
                              3         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
; Commentout below three lines
;@      IN      NS      localhost.
;@      IN      A       127.0.0.1
;@      IN      AAAA    ::1

;Name Server Information

@       IN      NS      ns1.rolandw.lan.

;IP address of Name Server

ns1     IN      A       10.0.0.10
```

```output
// /etc/bind/zones/reverse.rolandw.lan.db
;
; BIND reverse data file for local loopback interface
;
$TTL    604800
@       IN      SOA     rolandw.lan. root.rolandw.lan. (
                              3         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
;@      IN      NS      localhost.
;1.0.0  IN      PTR     localhost.

;Name Server Information
@       IN      NS     ns1.rolandw.lan.

;Reverse lookup for Name Server
10      IN      PTR    ns1.rolandw.lan.
```

### Additional debugging

If things dont seem to work you can try these things.

#### Issue: nameserver is 10.0.0.1 (incorrect nameserver assigned to clients)

By default most routers like to give out the DNS (aka nameserver) as itself. In this case the default gateway of the 10.0.0.0/24 subnet is 10.10.10.1 so thats what a DHCP provisioned device will get. To fix this you should change the DHCP settings on your DHCP server to hand out the correct IP

#### Issue: public DNS queries stop working

This is normally caused by Not enough DNS servers or no recursion feature implemented for fallback nameservers. If in your clients `/etc/resolv.conf` 10.0.0.10 is your only source of DNS then a client has no way of being able to reach the public internet.

To solve this you can implement recursion as described above in "Making bind into a recursive query server".
