# Ubiquiti Log

A "log/personal note style" journal of how i configured and set up my Ubiquiti network equipment.

Notes are in no particular order and also discuss off topic non Ubiquiti related areas of interest that i found useful when working towards my goals.

#### The goal

Set up my home network and learn new networking stuff through hands on practice!

## Network structure

My network will consist of these pretty basic key components:

* PFSense router running on an old SFF computer.
* Ubiquiti Unifi AP to provide a WLAN on 802.11ac to connect most of my devices
* Ubiquiti Edgeswitch (x10 ports) to create a LAN
* Netgear WN3000RPv3 range extender to serve devices on the other end of the house on 802.11n

## PFSense

PFsense is a straightforward install when you have 2 nics in your router. I have experimented with using a PFSense box with just one nic and found it impossible to pass the wan IP from the bridge -> switch -> router

## NBN

Theres some things to note about NBN and its compatibility with PFSense. On the most part 90% of ISPs can support PFSense, however sometimes your ISP has a sticky mac address for your router and wont recognize your new PFSense box.

As always ymmv and theres no gurentee as the lackey ISP rep servicing/diagnosing your connection is trained to blame any issues on PFSense as they are not trained and its not officially supported. On Telstra HFC it is 100% viable however.

The following technologies are used on NBN and should be researched before implementing your router. [source](https://whirlpool.net.au/wiki/modem_to_router_bridging_guide)

**Legacy (Copper):**

* Protocol: ADSL2+
* Who supplies **modem** feature? You
* Who supplies **router** feature? You

**FTTN (copper):**

* Protocol: VDSL2
* Who supplies **modem** feature? You
* Who supplies **router** feature? You

**FTTC (copper):**

* Protocol: VDSL2 (then G.FAST or XG.FAST)
* Who supplies **modem** feature? NBNco
* Who supplies **router** feature? You

**FTTB (copper):**

* Protocol: VDSL2 (then G.FAST or XG.FAST)
* Who supplies **modem** feature? You
* Who supplies **router** feature? You

**FTTP:**

* Protocol: GPON
* Who supplies **modem** feature? NBNco
* Who supplies **router** feature? You

**HFC:**

* Protocol: DOCSIS 3.0 (Then 3.1)
* Who supplies **modem** feature? NBNco
* Who supplies **router** feature? You

**Fixed Wireless:**

* Protocol: ?
* Who supplies **modem** feature? NBNco (outdoor antenna and NBN connection box).
* Who supplies **router** feature? You.

**Satellite:**

* Protocol: ?
* Who supplies modem feature? NBNco
* Who supplies router feature? You.

#### ISP Router authentication

Multi-Technology-Mix (NBN MTM) has resulted in a variety of different authentication methods that are used across a variety of Retail Service Providers (RSPs). [source](https://whirlpool.net.au/wiki/vdsl2_modem_routers_isp_settings)

Sometimes your ISP may tag your WAN traffic with a vlan ID, in other cases it may be untagged or vlan 1 traffic which is the best case scenario as you will probably need no further config.

For my network using Telstra it was very straight forward and was 100% zeroconf!

##### Telstra / Telstra Business

* Protocol: IPoE (DHCP / Automatic IP)
* VLAN: None/blank
* Login: No login required

Some other big ISPs are below

##### TPG

* Protocol: PPPoE
* VLAN ID: 2
* MTU: auto or 1492
* login: as provided

##### Optus

* IPoE (DHCP)
* No login required
* First connection must be done with Optus supplied modem
* After that, can switch to BYO modem but MUST copy Optus Modem VDSL MAC Address to your modem MAC – Most modems support this feature. (Certainly the el-cheapo TP-Link TD-W9970 does)

##### Kogan

* Protocol: IPoE (DHCP / Dynamic IP)
* MTU: auto or 1500
* No Login Required

IINet
[check here](https://help.iinet.net.au/n/iinet-broadband-settings-list)

#### Setting up PPPoE link for PFSense

Source to oreilly PFSense 2 cookbook [here](https://www.oreilly.com/library/view/pfsense-2-cookbook/9781849514866/ch07s03.html)

1. Browse to Services | PPPoE Server.
2. Click the "plus" button to add a new PPPoE instance.
3. Check Enable PPPoE Server.
4. Choose an Interface.
5. Choose a Subnet Mask.
6. Set No. PPPoE Users to the maximum number of clients we wish to allow.
7. Set Server Address to an unused IP address that pfSense will use to serve PPPoE clients.
8. Set Remote Address Range to the starting unused IP address. The range will
run as far as the maximum number ...

#### General Additional notes

9. Set your VLAN ID if required by your ISP
10. Enable DHCP on your WAN port to receive the WAN address
11. Set your LAN ip and network subnet (eg 192.168.0.1/24 - or leave on pfsense default)

## UniFi Controller Install Process

### Setting up Unifi on Arch

Linux has become my daily driver for the past 8 months, and while i plan on one day intergrating a unifi controller on the network on a physical device it is not required for operation of an Access Point.

To install is very similar on all distributions. I am using Arch and the [unifi](https://www.google.com/search?client=firefox-b-d&q=unifi+aur) package is available on the aur. I suggest the stable branch, not the beta one for obvious reasons.

```none
yay -S unifi
```

Once its installed start and enable the service.

```none
sudo systemctl enable unifi
sudo systemctl start unifi
sudo systemctl status unifi
```

By default you can now navigate to the unifi web interface at [localhost:8443](https://localhost:8443). Create an account and follow the set up instructions. The unifo controller installation docs can be found [here](https://dl.ubnt.com/guides/UniFi/UniFi_Controller_V5_UG.pdf) from chapter 1 (first couple pages).

Your AP should be available under the devices menu and can be provisioned accordingly. If your AP is not being detected here are some debugging tips. **Note** that ubiquity edge switches from experience switches are not part of the unifi control panel experience and their set up is covered below.

#### Debugging UniFi install

##### AP Power on

Is the AP turned on? Does it have enough power? Make sure the PoE injector is plugged into the AP and the ring light is blue on the top of the device

##### Correct Ports on PoE Injector

Is the PoE injector ports set up correctly? Make sure that you have **injector PoE -> AP** and **Injector LAN -> Switch**

##### Ping Connectivity to AP

Can you detect the AP on the network through ping? Try nmap on your network or go to your router and look at its ARP table or DHCP leases. Note that i dont believe that the unifi AP comes with SSH enabled to it out of the box. It can be enabled through the UniFi network controller however (though it needs to detect the AP first so its a null point in debugging)

## Configuration

### Enable SSH

A good idea (depending on who you ask) is to enable SSH access to your devices. You of course are sacrificing security by doing this, but thats the price you pay for convenience and shell access for future debugging.

A reason why this might be important to enable is on a temporary occasion when an access point needs a firmware update and it refuses to update itself automatically.

**To enable SSH**. Go to settings -> site -> and tick *"Enable Advanced Features"* under the Services tab.

**To set the password**. Scroll down the same page (settings/site) and change the authentication details under Device Authentication.

Also you can optionally add your SSH key which is a good idea. SSH keys are applied **site wide** meaning that your key will work anywhere on that site on any AP.

Now SSH into the device over the command line using the username configured above.

```none
ssh -p 22 RolandIRL@192.168.0.11 -i ~/.ssh/id_rsa
			^			^				^
			|			|				|
			Username	AP Address		Optional SSH key location
```

You can also configure a host entry for your AP for even faster future access in `~/.ssh/config`.

```none
host UniFi_AP
	User RolandIRL
   	Hostname 192.168.0.11
  	port 22
	IdentityFile /home/roland/.ssh/id_rsa
```

```none
BusyBox v1.25.1 () built-in shell (ash)


  ___ ___      .__________.__
 |   |   |____ |__\_  ____/__|
 |   |   /    \|  ||  __) |  |   (c) 2010-2020
 |   |  |   |  \  ||  \   |  |   Ubiquiti Networks, Inc.
 |______|___|  /__||__/   |__|
            |_/                  https://www.ui.com/

      Welcome to UniFi UAP-AC-LR!
```

## Configuring Edge Switch

Edge routers **DO NOT** use the ubiquiti UniFi controller software. You need an 💸💸💸 UniFi Switch instead (the white ones that cost a lot)

1. Plug in the switch and turn on power. Wait a min for it to start up
2. Identify the switch on the network (nmap or ARP table analysis or DHCP leases)
3. SSH is not enabled by default so instead the edge switch hosts a web server. you need to navigate to the IP in a browser ([https://192.168.x.y](https://192.168.x.y))
4. Log in with Ubnt/Ubnt and change the password to something strong
5. You should now have web access to the switch

### Edge Switch Configuration

Enable SSH in the web UI. You can now SSH into the EdgeOS shell which is very similar to Cisco IOS, you will feel right at home here if you have had any experience here. Use the username ubnt and the password you just set (otherwise the default pass is ubnt)

Unfortunately my particular edge switch does not appear to support RSA keys. So you will need to log in with a password (same password as the web interface).

```none
ssh ubnt@192.168.0.15
```

#### Change the username

SSH into the switch and perform the following commands.

```none
# config
# username myUsername secret myPassword
```
