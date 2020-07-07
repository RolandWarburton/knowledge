# Squid

From the squid [website](http://www.squid-cache.org/)

> Squid is a caching proxy for the Web supporting HTTP, HTTPS, FTP, and more. It reduces bandwidth and improves response times by caching and reusing frequently-requested web pages. Squid has extensive access controls and makes a great server accelerator. It runs on most available operating systems, including Windows and is licensed under the GNU GPL.

## Forward

For the purposes of this note i am only covering how to use squid as a http/https proxy. Squid can be used for much more than that but i haven't tried these new features out yet.

## Installing squid proxy on Debian 10 Buster

### Install squid package

Squid is in the standard repos so install using apt.

```none
sudo apt update
sudo apt install squid
```

### configure squid

Edit the `/etc/squid/squid.conf` file. Its good practice to make a backup first `sudo cp /etc/squid/squid.conf /etc/squid/squid.conf.old`

Check that squid is running

```output
● squid.service - Squid Web Proxy Server
   Loaded: loaded (/lib/systemd/system/squid.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2020-07-07 02:57:46 UTC; 2s ago
     Docs: man:squid(8)
  Process: 5126 ExecStartPre=/usr/sbin/squid --foreground -z (code=exited, status=0/SUCCESS)
  Process: 5129 ExecStart=/usr/sbin/squid -sYC (code=exited, status=0/SUCCESS)
 Main PID: 5130 (squid)
    Tasks: 3 (limit: 1149)
   Memory: 15.5M
   CGroup: /system.slice/squid.service
           ├─5130 /usr/sbin/squid -sYC
           ├─5132 (squid-1) --kid squid-1 -sYC
           └─5133 (pinger)
```

Next reference these following configs.

#### Authenticate with a password

Make sure your config has these lines referencing a passwd file

```none
# Point to a password
auth_param basic program /usr/lib/squid3/basic_ncsa_auth /etc/squid/htpasswd
auth_param basic realm proxy
acl authenticated proxy_auth REQUIRED

# Allow these things
http_access allow localhost
http_access allow authenticated

# Deny these things
http_access deny all
```

Next generate the before mentioned `/etc/squid/htpasswd` file using the following command. Make sure to replace **MYUSERNAME** and **MYPASSWORD**.

```none
printf "MYUSERNAME:$(openssl passwd -crypt 'MYPASSWORD')\n" | sudo tee -a /etc/squid/htpasswd
```

#### Authenticate by your ip

The easy way to do this is using an ACL inside the `/etc/squid/squid.conf` file.

```none
# Allow everything with the ip add 11.22.33.44 to authenticate
acl myNetwork 11.22.33.44
http access allow myNetwork
```

#### Example full config

Use the following config to copy-paste into your squid.conf to test with.

```none
# port of proxy
http_port 3128

# hostname of proxy
visible_hostname SuperProxy

# IP header HTTP masking (X-Forwarded-For: unknown)
forwarded_for off

# logs
access_log /var/log/squid/access.log
cache_log /var/log/squid/cache.log

# no cache
cache deny all

# DNS servers
dns_nameservers 8.8.8.8 8.8.4.4

# DNS cache
positive_dns_ttl 5 minutes
negative_ttl 5 minutes

# no wait before close Squid (30 seconds else, use of cache if enabled)
shutdown_lifetime 0 seconds

# allows a specific IP address
#acl myNetwork src 11.22.33.44
#http_access allow myNetwork

auth_param basic program /usr/lib/squid3/basic_ncsa_auth /etc/squid/htpasswd
auth_param basic realm proxy
acl authenticated proxy_auth REQUIRED

http_access allow localhost
http_access allow authenticated

http_access deny all
```

### Configure your browser

Now that you have squid configured, you need to tell your browser to use the proxy.

On firefox navigate to preferences -> proxy settings and configure as follows.

![FF proxy settings](https://i.imgur.com/xYEnyk9.png)

Make sure to add an exclusion for your local LAN. If you dont you will be unable to connect to http/https web interfaces that are hosted on your local network.

I think this is because your browser will try and route the internal traffic OUTSIDE the network, through the proxy, and then back INSIDE your network at which point the packed will most likely be dropped to a no_bogon rule on your router that prevents private packets being routed if they come from outside the network. See [Bogon filtering](https://en.wikipedia.org/wiki/Bogon_filtering).

### Additional resources

1. The article that i read to create these notes - [here](https://linuxize.com/post/how-to-install-and-configure-squid-proxy-on-debian-10/).
2. ServerFault question about proxies and local addresses - [here](https://serverfault.com/questions/55010/firefox-sends-local-traffic-to-proxy-server).
