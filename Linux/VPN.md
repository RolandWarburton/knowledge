# VPNs

### Connecting to Swinburne

Swinburne uses [Cisco Anyconnect](https://www.cisco.com/c/en_au/products/security/anyconnect-secure-mobility-client/index.html) Which is a flavour of VPN (an entire protocol as i understand) that is pretty much just like any identical software based VPN.

More notes on the types of VPNs are covered under [TNE30009 Network Security and Resilience](https://rolandw.dev/Notes/University/TNE30009NetworkSecurityandResilience/#week-6-lecture-vpns).

On some distros the cisco anyconnect client does not work and from my experience installing on arch i had the following error.

```output
rm: cannot remove '/etc/rc.d/vpnagentd': No such file or directory
mv: cannot stat '/opt/cisco/vpn/*.log': No such file or directory
install: cannot create regular file '/etc/rc.d/vpnagentd': No such file or directory
```

The solution to this is to not use the provided anyconnect client provided by cisco and instead use an open source alternative, enter [OpenConnect!](https://wiki.archlinux.org/index.php/OpenConnect).

OpenConnect can be installed using your distros package manager.

On arch based systems.

```none
pacman -S openconnect
```

On debian based systems.

```none
apt install openconnect
```

Then using OpenConnect is very straight forward. Log in with your student ID (102111111) and password when prompted.

```none
sudo openconnect vpn.swin.edu.au
```

The command will then appear to freeze. This means that you are connected. You can now minimize that terminal window. Alternatively if you want to you could use a multiplexer like tmux or screen to continue using the terminal.

There also exists a [networkmanager-openconnect](https://www.archlinux.org/packages/extra/x86_64/networkmanager-openconnect/) package that allows network manager to work with OpenConnect however i have not gotten this working yet (will update if i ever do get it working).

#### Debugging

Make sure your network manager of choice is working.

In the case of [NetworkManager](https://wiki.archlinux.org/index.php/NetworkManager) make sure the system is running.

```none
systemctl status NetworkManager
```

```output
NetworkManager.service - Network Manager
     Loaded: loaded (/usr/lib/systemd/system/NetworkManager.service; disabled; vendor preset: disabled)
    Drop-In: /usr/lib/systemd/system/NetworkManager.service.d
             └─NetworkManager-ovs.conf
     Active: active (running) since Tue 2020-08-04 20:26:19 AEST; 1h 19min ago
       Docs: man:NetworkManager(8)
   Main PID: 5665 (NetworkManager)
      Tasks: 3 (limit: 19144)
     Memory: 7.1M
     CGroup: /system.slice/NetworkManager.service
             └─5665 /usr/bin/NetworkManager --no-daemon
```
