# PFSense

Log of PFSense notes.

## PFBlockerNG

Make sure your DNS servers on linux are actually pointing to your PFSense router.

```bash
cat /etc/resolv.conf
```

Check to see if a domain is blocked or not by using the `drill` command from the ldns package.

The result of a non blocked website will return its correct IP from its domain name.

Example of successful domain resolution (domain **isnt** blocked)

```bash
drill isitblocked.org
```

```bash
;; ANSWER SECTION:
isitblocked.org.	3126	IN	A	74.208.236.124
```

Example of unsuccessful domain resolution (domain **is** blocked).

The returned IP is the virtual IP of PFBlockerNG.

```bash
drill analytics.163.com
```

```bash
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
