# Networking Part 4 - IP Addresses

## IP Address Cheat Sheet

### Set a static IP

Edit `/etc/network/interfaces` as root. Make sure to change the name of the interface to the interface that you want to change.
To find the possible names of interfaces run `ip add`.

```none
auto ens33
iface ens33 inet static
	address 192.168.0.50
	netmast 255.255.255.0
	network 192.168.0.0
	broadcast 192.168.0.255
```

Then flush and restart your IP. You might need to unplug and plug your ethernet cable to speed up the dhcp request process.

```none
sudo ip addr flush ens33  && sudo systemctl restart networking
```
