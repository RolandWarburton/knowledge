# X11 Forwarding
X11 Forwarding allows you to launch **applications** from remote linux systems on your system. Think screenshare but for a single application (It is possible to share an entire DE but i have never attempted it).
```
+--------+>--------file.txt------->+--------+
| SERVER |                         | CLIENT |
+--------+<-------ssh-server------<+-+----+-+
└── file.txt                         ^    |
                                     |    |
                                     |    v
                             +-------+----+------+
                             | File.Txt          |
                             |                   |
                             | text text text    |
                             | text text text    |
                             | text text text    |
                             |                   |
                             |                   |
                             |                   |
                             |                   |
                             +-------------------+
```
1. install *xauth* and optionally *xclock* for testing.
2. set $DISPLAY variable to ```localhost:displaynumber.screennumber```. Example: ```export DISPLAY=localhost:10.0```.
3. edit ```/etc/ssh/sshd_config``` and ```/etc/ssh/ssh_config```.
```
# /etc/ssh/sshd_config
#
# Either...
# Globally
X11Forwarding yes
X11DisplayOffset 10 <- this is the display number used in step 2.
#
#Per user/group
Match user roland
X11Forwarding yes
X11DisplayOffset 10
```
```
#/etc/ssh/ssh_config
#
Host *
   ForwardX11 yes
   ForwardX11Trusted yes
```
Connect to the server with the -X tag (-X is not needed if you set up sshd_config X11Forwarding) ```ssh -p 22 -X roland@server```
* Observe xauth to see that a key was created for forwarding ```xauth list $DISPLAY```. You may need to add this key with ```xauth add localhost.displaynumber:10 MIT-MAGIC-COOKIE-1 4d22408a71a55b41ccd1657d377923ae```

### Optimising X11 Forwarding
By default X11 Forwarding is pretty slow.
* Use the -Y tag to indicate you trust the machine instead of the -X tag. This will bypass security checks and might make the connection faster.
* use the -C tag to compress the connection. 
* prevent timeout by editing ```~/.ssh/config``` and adding ```ForwardX11Timeout 1d``` (1 day).
