# NAT Basics

Having lots of trouble with WG and i thought it was a NAT issue, so i took these notes. Maybe one day i will get it working.

**Edit**: So turns out it wasn't a NAT problem per se, but these notes are still useful.

## NAT

### 1:1 NAT

1:1 NAT is maps **one** IP4 address (public) to **one** IP4 address (private), and vice versa.

* In a scenario where you have a webserver...
* you own a second public IP address
* you use 1:1 NAT to map anything from your webservers **private** IP 192.168.0.1 to your **public** IP address that specially exists just to serve webserver traffic.

Likewise all traffic coming from the **public** to your **private** IP will be translated to 192.168.0.1.

The traffic from **public** space will be translated to **private** space, then evaluated by the **firewall** WAN rules.

1:1 can also translate an entire subnet but this is not common because you need as many external public addresses as you have internal private ones.

Port translations on 1:1 NAT when traffic is outbound is constant.

* IE Traffic from 192.168.0.1:8000 is sent to the client on 1.2.3.4:8000
* If a client requests a website on 1.2.3.4:80 then it will get a response on :80

### Inbound and Outbound NAT

Inbound NAT, otherwise known as Port Forwarding, is used to direct traffic to a specific internal address.

For example a user requests a website from your server on your public ip 1.2.3.4:80, the inbound NAT rule will translate this to your internal server on 192.168.0.1:80.

Outbound NAT works the reverse of Inbound NAT.

For example when the web server is replying to requests, it knows that the port it arrived on was 80 and the address it came from was 192.168.0.1 (the routers address). When the reply reaches the router, the router may apply an outbound NAT rule based on the source of the web server and translate the address of the packet to a different address.

### Outbound NAT with Wireguard

As we discussed. Outbound (or Source) NAT translates the **source** address and port when it is **leaving** the interface.

Therefore in normal traffic (think of a web server)

* the client requests the page from 1.2.3.4:80
* the router receives a request for 1.2.3.4:80 and forwards it to 192.168.0.1:80. (outbound NAT rule translates it to the wan interface as the source IP)
* The web server receives and responds with a source of 192.168.0.1:80
* the router receives the response and by default NAT translates the source 192.168.0.1:80 back into 1.2.3.4:80

<!-- What we need to do is to change the source address of all packets that come from 10.10.0.0/24 (wireguard network) to that of the wireguard interface address.

I will create an outbound NAT rule similar to the default IKE outbound nat rule that is as follows:

* source: 127.0.0.1/8 (localhost, /8 is the SN of localhost)
* destination: any
* destination port: 51820
* translation address: interface address (connections will be mapped to whatever the interfaces address is)

Then repeat for source 10.10.0.0/24 (wg network) and 10.0.0.0/24 (server network) -->
