# FreeRADIUS

## Setup

This guide is for CentOS 8 but most likely also works on RHEL 8 and other enterprise linux distros.

### Step 1. Install packages

```none
dnf install freeradius*
```

```none
# or to install just freeradius
dnf install freeradius
```

The config files for freeradius can be found in `/etc/raddb/*`.

| File         | Description                                                                      |
|--------------|----------------------------------------------------------------------------------|
| clients.conf | Networking Equipment. APs, Switches, etc. Clients must have a static IP address. |
| users        | User accounts who authenticate through a client.                                 |

### Step 2. Creating users

Its a good idea to backup your users before modifying the users file

```none
cp /etc/raddb/users /etc/raddb/users.bak
```

Once you have backed up, edit the users file `vim /etc/raddb/users`. Set the line numbers so you can follow these steps easier. `:set number`.

On line 87 there is an example user. Create a new user under the example user.

```none
roland Cleartext-Password := "rhinos"
```

### Step 3. Creating clients

Its a good idea to backup your clients before modifying the clients file

```none
cp /etc/raddb/clients.conf /etc/raddb/clients.conf.bak
```

Once you have backed up, edit the users file `vim /etc/raddb/clients.conf`.

In the clients file there is a testing client called `client localhost {...}`that you should read, dont delete this client.

Create a new client to use with one of the following methods... **Note that its preferred to use method 1**.

#### Create client. Method 1

```none
client myAccessPoint {
	ipaddr = 192.168.0.50
	secret = radiusPassword
}
```

#### Create client. Method 2

```none
client 192.168.0.50 {
	secret = radiusPassword
	shortname = myAccessPoint
}
```

### Step 3. Test FreeRADIUS authentication

Now that you have a user created you should start the RADIUS service in debugging mode using the -X flag.

```none
radiusd -X
```

Then try and authenticate as a user. A successful authentication will return an `Access-Accept`, otherwise check the radiusd debug log for a potential reason.

1. Run `radiusd -X` from the radius server
2. Open a new terminal and ssh into the radius server again
3. from the new terminal run the `radtest username password 127.0.0.1 0 HelloWorld` command below for your created user

```none
radtest roland rhinos 127.0.0.1 0 HelloWorld
```

If you get an `Access-Accept` back then its successful! According the the [FreeRADIUS getting started guide](https://wiki.freeradius.org/guide/Getting-Started) the following authentication methods will work.

```none
PAP, CHAP, MS-CHAPv1, MS-CHAPv2, PEAP, EAP-TTLS, EAP-GTC, EAP-MD5.
```

### Step 4. Authenticating through a client

In the above step we performed testing directly on the radius server, and without using a client. Next we will add a client and run FreeRADIUS authentication as a systemd service and authenticate a real **user** (my mobile) through a **client** (my unifi AP).

Start by creating a client in `/etc/raddb/clients.conf`.

```none
Client ubnt-ap {
	ipaddr = 192.168.0.50
	secret = myPassword
}
```

From the [FreeRADIUS getting started guide](https://wiki.freeradius.org/guide/Getting-Started)...

You should change the IP address **192.168.0.50** to be the address of the client which will be sending **Access-Request** packets.

The client should also be configured to talk to the RADIUS server, by using the IP address of the machine running the RADIUS server. The client must use the same secret as configured above in the **client** section (the above codeblock).

Then restart the server in debugging mode, and run a simple test using the **roland** user. You should see an **Access-Accept** in the server output.

### Step 4.5. Adding RADIUS server to the AP config

In my real life practice i am using an AP as a **client** to authenticate **users** with.

* The IP to the AP is `192.168.0.50`
* The IP of the RADIUS server is `192.168.100.10`

Use the following screenshots as a general guide.

![Adding the RADIUS server](https://i.imgur.com/7qa9LWp.png)

![Adding the RADIUS server](https://i.imgur.com/NAFYQb7.png)

![Adding the RADIUS server](https://i.imgur.com/cJrk3at.png)

![Adding the RADIUS server](https://i.imgur.com/Ikl4YPp.png)

### Step 5. Firewall configuration

In my testing i experienced many headaches with firewall configuration. I believe it boils down to the firewall that comes with CentOS 8 being enabled by default and blocking RADIUS (port 1812) by default.

The firewall that comes with CentOS 8 and other enterprise distributions of linux is firewalld, digital ocean has a great article about configuring firewalld [here](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-using-firewalld-on-centos-7).

I found these settings to work pretty well for me.

First check that firewalld is running and have a look at its zone configuration. Firewalld uses zones to control traffic on different interfaces, these interfaces can be physical or virtual. 

```none
# Have a look at the state of the firewall
firewall-cmd --state
```

As i am running my RADIUS server as a VM on ESXi 6.7 i have an additional internal zone and interface [libvirt](https://en.wikipedia.org/wiki/Libvirt) and virbr0 which are added for VM networking support. The "virbr0" is a bridge interface that represents a network (a bridge is essentially a network switch) that connects the VM to the virtual switch.

```none
# Check that the correct interface is being used and that the public zone is enabled
firewall-cmd --get-active-zones
```

```output
libvirt
  interfaces: virbr0
public
  interfaces: ens192
```

```none
# Inspect the public zone to see which services are enabled for this interface
firewall-cmd --zone=public --list-all
```

```output
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens192
  sources:
  services: cockpit dhcpv6-client ssh
  ...
```

```none
# Inspect the libvirt zone to see which services are enabled for this interface
firewall-cmd --zone=libvirt --list-all
```

```output
  target: ACCEPT
  icmp-block-inversion: no
  interfaces: virbr0
  sources:
  services: dhcp dhcpv6 dns ssh tftp
  ...
  rich rules:
	rule priority="32767" reject
```

Notice how these zones are now allowing the radius service or the radius port. This means that packets destined for the RADIUS server on its public and vitalized bridge interface will be blocked. This can be confirmed by running a packet capture on the RADIUS network (192.168.100.0) interface from the router.

I have obtained the following screenshot from my routers 192.168.100.0 interface that shows the RADIUS packets being denied.

![packet capture radius being blocked](https://i.imgur.com/aAr8Hyd.png)

Lets add a rule to the RADIUS host that allows radius traffic using the following commands.

```none
# add the radius service permanently to the public zone to persist over reboots
firewall-cmd --zone=public --permanent --add-service=http
```

```none
# add the radius service permanently to the libvirt zone to persist over reboots
firewall-cmd --zone=libvirt --permanent --add-service=http
```

Now try to authenticate to RADIUS again with a user through the client and check to see if it works. This solved the packet blocked issues that i was having when testing.

If there are still problems with packets being blocked you can try temporarily disabling the firewalld service to remove the FW. **This will make your radius server vulnerable!!! ONLY do this for testing**.

```none
# Check that firewalld is running
firewall-cmd --state
```

```output
running
```

```none
# Disable the firewall
systemctl stop firewalld
systemctl disable firewalld

# Prevent another service from starting the firewall service
# Map the service to /dev/null
systemctl mask --now firewalld
```

```none
# To unmask the service (once finished testing)
systemctl unmask --now firewalld
```

### Logging

Logging is enabled in `/etc/raddb/radiusd.conf`.

Create a backup of the config file first.

```none
cp /etc/raddb/radiusd.conf /etc/raddb/radiusd.conf.bak
```

Next edit the config file and change auth to `auth=yes` on line 304.

```none
vim /etc/raddb/radiusd.conf
# :304 -> change to yes
# :312 -> change to yes to log failed auths
# :313 -> change to yet to log success auths
```

Log files will be stored in the following location.

```none
less /var/log
```

An example of a log event from the user **roland** logging in through client **ubnt-ap**.

```output
Sat Jul 18 19:07:29 2020 : Auth: (32) Login OK: [roland/<via Auth-Type = eap>] (from client ubnt-ap port 0 cli 22-FC-93-19-0D-EF)
```
