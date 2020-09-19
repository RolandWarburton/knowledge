# PFSense

Log of PFSense notes.

## PFBlockerNG

Make sure your DNS servers on linux are actually pointing to your PFSense router.

```none
cat /etc/resolv.conf
```

Check to see if a domain is blocked or not by using the `drill` command from the ldns package.

The result of a non blocked website will return its correct IP from its domain name.

Example of successful domain resolution (domain **isnt** blocked)

```none
drill isitblocked.org
```

```none
;; ANSWER SECTION:
isitblocked.org.	3126	IN	A	74.208.236.124
```

Example of unsuccessful domain resolution (domain **is** blocked).

The returned IP is the virtual IP of PFBlockerNG.

```none
drill analytics.163.com
```

```none
;; ANSWER SECTION:
analytics.163.com.	59	IN	A	10.10.10.1
```

## Captive Portal

Captive portal can be very straight forward to set up.

Start by navigating to `Services -> Captive Portal` and Createing a new one by clicking `Add` and giving it any name and description.

Once created, tick the enable button to see the full list of options

1. Select an interface to apply this to. Be aware that initially devices on this interface will have trouble connecting to the internet, however they should still be able to access the routers address (the IP of my router is 192.168.0.1)
2. Tick `Use custom captive portal page` and Upload a html file under `Portal page contents`

Example of a custom portal page.

```html
<!DOCTYPE html>
<html lang="en">

<head>
	<meta charset="UTF-8">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<title>login</title>
</head>

<body>
	<form method="post" action="$PORTAL_ACTION$">
		<input name="auth_user" type="text" placeholder="name">
		<input name="auth_pass" type="password" placeholder="password">
		<input name="auth_voucher" type="text" placeholder="voucher">
		<input name="redirurl" type="hidden" value="$PORTAL_REDIRURL$">
		<input name="zone" type="hidden" value="$PORTAL_ZONE$">
		<input name="accept" type="submit" value="Continue">
	</form>
</body>

</html>
```

3. Next give any special devices that are outside the DHCP range, but still on the same interface that the captive portal is on special access under `Allowed IP Addresses`.

For example my desktop PC is on my captive portal LAN interface, but outside the DHCP range. DHCP is between 192.168.0.10 - 192.168.0.50 and my PC is on 192.168.0.100, so i add 192.168.0.100 as allowed in both directions.

Another common example is if you are running services like PFBlockerNG that perform DNS filtering, by default the ip address that PFBlockerNG uses as its DNS is `10.10.10.1`. So you need to add this address to the list of allowed IPs (both directions as well) for DNS resolution to work on the captive portal.

More useful debugging information for the captive portal can be found [here](https://docs.netgate.com/pfsense/en/latest/captiveportal/captive-portal-troubleshooting.html).

### Captive portal user authentication

First create a new group under `System -> User Manager -> groups`. Call the group **Captive_Portal** and give it local scope. Next click the *Add* button on the same page and assign the group Captive_Portal group the **User - Services: Captive Portal login** permission.

Now go back to users and create a new user, Make sure to give them a password AND add give them group membership of the Captive_Portal group.

### Captive portal vouchers

Under `Services -> Captive Portal` click edit on your captive portal and then navigate to the vouchers tab.

From the vouchers tab click the enable button and then click save.

Now scroll back up to the top of the vouchers page and you should be able to click the `Add` button to create a new *Voucher Roll*

You can give the roll any number you choose, if you give the new roll the number of a previous roll the old one will be overwritten and delete any unactivated/activated vouchers.

Next pick the minuites per voucher, count (number of vouchers to generate in this particular roll), and an administrative comment.

You can now go back to the main vouchers tab and click the export button on the row for the rolls you just generated.

To test your vouchers copy one from the exported csv and go to `Status -> Captive Portal` and navigate to the `Test Vouchers` tab and paste in a voucher to see its elegibility and time remaining in the database.

### Captive portal custom login page

The captive login page is located at `/var/etc` on the router. To get there, ssh into the router and select option 8 (shell). To copy files you can sftp to your own computer once connected and move images etc over.

Here is an example of a decent looking captive portal page.

```html
<!DOCTYPE html>
<html lang="en">

<head>
	<meta charset="UTF-8">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<title>login</title>
</head>

<body>

	<div id="form-wrapper">
		<form method="post" action="$PORTAL_ACTION$">
			<div id="login-box">
				<input name="auth_user" type="text" placeholder="Name">
				<input name="auth_pass" type="password" placeholder="Password">
			</div>
			<div id="voucher-box" style="display: none;">
				<input name="auth_voucher" type="text" placeholder="voucher">
				<div id="voucher-info">
					<img src="./question.svg" alt="?" width="25px" height="25px">
					Vouchers are a single use token that enable access to the internet for a limited time.
				</div>
			</div>
			<input name="accept" id="submit" type="submit" value="Continue">
			<input name="redirurl" type="hidden" value="$PORTAL_REDIRURL$">
			<input name="zone" type="hidden" value="$PORTAL_ZONE$">
			<a href="#" onClick="toggleVoucher(this)" id="toggle-voucher">I have a voucher</a>
			<p id="version">v1.0</p>
		</form>
	</div>

	<style>
		body {
			background-color: #2F303A;
			font-family: 'Roboto', sans-serif;
			margin: 0;
			padding: 0;
		}

		* {
			margin: 0;
			padding: 0;
		}

		#form-wrapper {
			background-color: #585961;
			position: absolute;
			left: 50%;
			top: 50%;
			transform: translate(-50%, -50%);
			justify-content: center;
			margin: auto 0;
			box-shadow: 0 6px 10px black;
			height: 300px;
			width: 300px;
		}

		form {
			margin: 4ch;
		}

		form input {
			box-sizing: border-box;
			margin: 2ch 0;
			width: 100%;
		}

		#toggle-voucher {
			text-decoration: none;
			color: #dedede;
			position: absolute;
			padding: 1ch;
			bottom: 0;
			left: 0;
		}

		#version {
			position: absolute;
			bottom: 0;
			right: 0;
			padding: 1ch;
			margin: 0 auto;
			bottom: 0;
			color: #dedede;
		}
	</style>

	<script>
		const toggleVoucher = (event) => {
			const voucherBox = document.querySelector("#voucher-box");
			const loginBox = document.querySelector("#login-box");

			const voucherIsHidden = (voucherBox.style.display == "block") ? false : true;
			if (voucherIsHidden) {
				voucherBox.style.display = "block";
				loginBox.style.display = "none";
			} else {
				voucherBox.style.display = "none";
				loginBox.style.display = "block";
			}
		}
	</script>
</body>

</html>
```

## PFSense on a laptop

Ie. How to set up PFSense on a device with 1 port using vlans.

![pktTracer_Diagram](/media/PFSense_Diagram.png)

### Installing

1. Go to [the website](https://www.pfsense.org/download/) and download the latest version

```none
Version: 2.4.5
Architecture: AMD64 (64bit)
Installer: USB Memstick Installer
Console: VGA
Mirror: Singapore
```

2. Flash image to usb `sudo dd status=progress if=pfSense.img of=/dev/sd[drive]`
3. Follow the install instructions in the installer
4. If you are installing on a laptop (like me) you will likely only have 1 eth port on your laptop, so you will need a switch to facilitate your internet modem, computer and other devices, and your laptop pfsense router. I am using a Cisco 3750 and have connected F0/0/1 -> modem, and F0/0/2 -> laptop router.

### Setting up Basic Configuration

* My current router is on 192.168.0.0/24
* ISP/default gateway (telstra modem) = 192.168.0.1
* Switch LAN IP = 192.168.0.10
* Switch LAN vlan ID = 200
* PFSense WAN = re0.100	no ip
* PFSense LAN = re0.200 192.168.0.200

### Switch Config

```none
vlan 200
	name LAN

int vlan 200
	ip add 192.168.0.10 255.255.255.0

int f1/0/1
	description connection to internet
	switchport mode access
	switchport access vlan 200

int f1/0/3
	description connection to PFSense
	switchport trunk encapsulation dot1q
	switchport mode trunk

! An example interface that you would put a LAN PC on that reports to PFSense
int f1/0/3
	description "put a LAN PC here"
	switchport access vlan 200
	switchport mode access
```

### PFSense Config

1. Create Vlans -> yes
2. Create 2 vlans on your single internet for WAN and LAN
   1. re0.100 = WAN
   2. re0.200 = LAN

Proceed and have PFSense finish setting up the router, you will receive the following IPs

```none
WAN (wan)	-> re0.100	->
LAN (lan)	-> re0.200	-> 192.168.1.1
```

Select option 1 to **assign interfaces** and change re0.200 to an IP that exists on your current network. My current network is using 192.168.0.0/24 so i will change the IP to 192.168.0.200 which will become the new gateway and ip to access PFSense.

```none
WAN (wan)	-> re0.100	->
LAN (lan)	-> re0.200	-> 192.168.0.200
```

You should now be able to connect to 192.168.0.200 (pfsense) from a computer on the 192.168.0.0/24 network. Once logged in go through the setup process and set there values

* Primary DNS = 8.8.8.8
* Secondary DNS = 8.8.4.4
* Enable internal traffic, because the PFSense box is on the inside of the network

### Testing

* You should be able to ping 192.168.0.10 (switch) -> 192.168.0.1 (modem)
* 192.168.0.10 (switch) -> 192.168.0.200 (pfsense) and vice versa
* You should be able to access 192.168.0.200 (pfsense) from a computer connected anywhere on 192.168.0.200

### Setting up SSH to connect to switch

```none
username roland privilege 15 secret p@ssw0rd
ip domain-name Sw4.com
crypto key generate rsa modulus 2048
ip ssh version 2
line vty 0 15
	transport input ssh
	login local
```

### Setting up SSH to connect to PFSense

web interface -> system -> advanced -> Secure Shell.

* Enable Secure Shell = true
* Allow Agent Forwarding = true
* SSHd Key Only = Password or Public Key

### Resolve PFSense using named

After setting up named on my network to resolve some common VM names, i decided to add my pfsense to that list, however i ran into the "potential DNS rebind attack" warning when i set up
an A record for "pf" at its location.

To fix this i renamed the entry in named to "pfsense" which matches the name hostname of the pfsense box. If you want to change the name to something else you can change it under `system -> general`.

Heres a sample for the reverse and forward zones respectively. Just as a reminder.

* reverse: ip -> nameserver
* forward: nameserver -> ip

```sh
; Reverse zone
;
; Router
pfsense	IN	PTR	pf.lab.lan
```

```none
; Forward zone
;
; Router
pfsense		IN	A	192.168.0.1
```
