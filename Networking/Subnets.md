# Subnets

This section doesn't contain exhaustive information about how subnetting is done. More its a reference for someone who already gets the gist of subnetting.

## IPv4 vs IPv6

#### IPv4
* IPV4 is a 32 bit numeric address written in 4 parts (eg. 60.224.192.150)
* Each group of numbers represents an Octet 
* The range of an octet can be any number between 0-255
* There are 4,294,967,296 unique addresses

#### IPv6
* IPV6 is a new generation of IP addresses
* The reason it was created was from a lack of IP addresses
* IPV6 has far more combinations that IPV4
* IPV4 is a 32 bit numeric address vs IPV6 which is a 128 bit hexadecimal address
  * Eg. 2001:0db8:85a3:0000:0000:8a2e:0370:7334.
* There are 340 undecillion unique addresses

## Private Ip Address Ranges

| Class | IP Range                           | Subnet Mask   |
|-------|------------------------------------|---------------|
| A     | 10.0.0.0 to<br>10.255.255.255      | 255.0.0.0     |
| B     | 172.16.0.0 to<br>172.31.255.255    | 255.255.0.0   |
| C     | 192.168.0.0 to <br>192.168.255.255 | 255.255.255.0 |

## Public Ip Address Ranges

Decidedly less important that private IPs because you dont get a choice in picking these.

Checking if an IP is public or not can be hard to know without staring at these 2 tables. If you want to check if an IP to see if its public run it through [this](https://ipinfo.info/html/ip_checker.php) and if it returns a whois result then its public.

|   | First Octet Address | IP Range                            | Default Subnet Masks | Spread                                                                                               |
|---|---------------------|-------------------------------------|----------------------|------------------------------------------------------------------------------------------------------|
| A | 1-126               | 1.0.0.1 to 126.255.255.254          | 255.0.0.0            | <br>16 million hosts on 126 networks.<br>This class is for large organizations.                      |
| B | 128-191             | 128.1.0.1 to<br>191.255.255.254     | 255.255.0.0          | <br>65 thousand hosts on 16 thousand networks.<br>This class is used for medium sized organizations. |
| C | 192-223             | <br>192.0.1.1 to<br>223.255.254.254 | 255.255.255.0        | Supports 254 hosts on 2 million networks                                                             |