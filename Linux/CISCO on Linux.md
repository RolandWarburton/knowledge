# Connect to CISCO equipment through a terminal

To connect to CISCO equipment through serial you can use minicom.
To run minicom for CISCO equipment you need to configure your settings with `minicom -s`.

1. Ensure you are using the correct serial device.
List your devices with `ls /dev` and look for the correct device to use to communicate with. In my case my serial cables are rj45 -> usb. So i used /dev/ttyUSB0* as my serial device.
2. Have the correct serial speed. And
Turn Hardware Flow Control off in minicoms settings.

To save the config use the *Save setup as dfl* option. dfl = default.

Connect the the device by running `minicom`. quit the connection with `ctrl+a, q`.

### Setting up KexAlgorithms and Ciphers

I have found that sometimes new versions of openssh have trouble negotiating a link with my cisco switch due to legacy issues ([i think](https://unix.stackexchange.com/questions/340844/how-to-enable-diffie-hellman-group1-sha1-key-exchange-on-debian-8-0)).

I have added a host to my ssh config `~/.ssh/config` to offer some older accepted ciphers and key exchange methods which seems to have solved my problem

```none
 host CiscoSwitch4
    GSSAPIAuthentication yes
    KexAlgorithms=curve25519-sha256@libssh.org,
	ecdh-sha2-nistp256,ecdh-sha2-nistp384,
	ecdh-sha2-nistp521,
	diffie-hellman-group-exchange-sha256,
	diffie-hellman-group14-sha1 diffie-hellman-group1-sha1
    Ciphers aes128-ctr,aes192-ctr,aes256-ctr,aes128-cbc,3des-cbc
    User roland
    HostName 192.168.0.10
    Port 22
```
