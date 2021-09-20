# Monitor Bind9 with Zabbix

Based on [this](https://ixnfo.com/en/monitoring-bind9-in-zabbix.html) article by ixnfo.com.

## Modify bind to export statistics

Edit `/etc/bind/named.conf.options` and place this outside of any scope (do **not** put it insdide the options section)

Modify the first line of the block `inet 10.0.0.10...` to use your own IPs.

* 10.0.0.10 - The DNS server IP
* 192.168.0.100 - My workstation
* 10.0.0.60 - Zabbix server

```none
statistics-channels { 
     inet 10.0.0.10 port 8080 allow { 192.168.0.100; 10.0.0.10; 10.0.0.60; }; 
     inet 127.0.0.1 port 8080 allow { 127.0.0.1; }; 
};
```

Then restart bind.

```none
sudo systemctl restart bind9
```

For debugging see if your elected (allowed) computers can pull the xml statistics data.

From another machine that is not the DNS server install the following:

```none
sudo apt-get install xml2 curl
```

Then run the following command:

```none
curl http://10.0.0.10:8080/ 2>/dev/null | xml2 | grep -A1 queries
```

## Add parameters to zabbix agent

On the DNS servers zabbix agent configuration file add the following:

```none
sudo vim /etc/zabbix/zabbix_agentd.conf
```

```none
# The number of connections udp to DNS:
UserParameter=bind.net.udp,netstat -nua | grep :80 | wc -l
# Number of tcp connections to DNS:
UserParameter=bind.net.tcp,netstat -nta | grep :80 | wc -l
# Number of incoming and outgoing requests:
UserParameter=bind.queries.in[*],curl http://127.0.0.1:8080/ 2>/dev/null | xml2 | grep -A1 "/statistics/server/counters/counter/@name=$1$" | tail -1 | cut -d= -f2
UserParameter=bind.queries.out[*],curl http://127.0.0.1:8080/ 2>/dev/null | xml2 | grep -A1 "/statistics/views/view/counters/counter/@name=$1$" | tail -1 | cut -d= -f2
UserParameter=bind.queries.query[*],curl http://127.0.0.1:8080/ 2>/dev/null | xml2 | grep -A1 "/statistics/server/counters/counter/@name=Qry$1$" | tail -1 | cut -d= -f2
```

Then restart the agent on the DNS server.

```none
sudo systemctl restart zabbix-agent.service
```

Next you need to create a template that can support these new parameters.

## Create zabbix template

From the zabbix web interface create a new template like so.

1. Click on config/templates
2. Search for the `template zabbix agent`
3. Click on the passive `template zabbix agent`

![step 1](https://media.discordapp.net/attachments/744488560060792872/889454240903340042/unknown.png?width=810&height=286)

1. Click clone and give it a new name test for bind

Now there is a new template based on the basic zabbix one

![step 2](https://media.discordapp.net/attachments/744488560060792872/889454512530681866/unknown.png?width=810&height=422)

Click on the new template `test for bind`

![step 3](https://media.discordapp.net/attachments/744488560060792872/889454683255607296/unknown.png?width=810&height=395)

1. Click on items
2. Click on create item

![step 4](https://media.discordapp.net/attachments/744488560060792872/889454857583489024/unknown.png?width=810&height=245)

Fill out this information.

1. The name `test item`
2. The key from below
3. Give it a description

Then click save and then go back to hosts.

![step 5](https://media.discordapp.net/attachments/744488560060792872/889455046960492564/unknown.png?width=810&height=608)

Click on items.

1. Search for the item
2. Select it
3. Execute it to grab data

![step 6](https://media.discordapp.net/attachments/744488560060792872/889455494815694868/unknown.png?width=810&height=231)

![step 7](https://media.discordapp.net/attachments/744488560060792872/889455647328989194/unknown.png?width=810&height=347)

It will appear in the latest data tab.

![step 8](https://media.discordapp.net/attachments/744488560060792872/889455934185811998/unknown.png?width=810&height=283)

Double check the values are correct by basing it against the DNS servers own stats page at [http://dns_server_ip:8080/](http://dns_server_ip:8080/).

![step 9](https://media.discordapp.net/attachments/744488560060792872/889456179200262144/unknown.png)

Examples of keys you can use.

```none
bind.queries.in[A]
bind.queries.out[A]
bind.queries.in[AAAA]
bind.queries.out[AAAA]
bind.queries.in[NS]
bind.queries.out[NS]
bind.queries.in[MX]
bind.queries.out[MX]
bind.queries.in[PTR]
bind.queries.out[PTR]
bind.queries.in[SOA]
bind.queries.out[SOA]
bind.queries.in[TXT]
bind.queries.out[TXT]
bind.queries.in[ANY]
bind.queries.out[ANY]
bind.queries.in[SPF]
bind.queries.out[SPF]
bind.queries.in[CNAME]
bind.queries.out[CNAME]
bind.queries.in[DS]
bind.queries.out[DS]
bind.queries.in[DNSKEY]
bind.queries.out[DNSKEY]
bind.queries.in[RRSIG]
bind.queries.out[RRSIG]
...
bind.queries.query[AuthAns]
bind.queries.query[Dropped]
bind.queries.query[Duplicate]
bind.queries.query[Failure]
bind.queries.query[FORMERR]
bind.queries.query[NoauthAns]
bind.queries.query[NXDOMAIN]
bind.queries.query[Nxrrset]
bind.queries.query[Recursion]
bind.queries.query[Referral]
bind.queries.query[SERVFAIL]
bind.queries.query[Success]
```
