# Networking (Part One)
### FTP Vs SFTP Vs SSHFS
* FTP: Comes in the form of vsftpd. I use this on my VPS because its easier to set up and easier (but not as secure) to connect to.
* SFTP: Comes in the form of OpenSSH. I use this on my main arch machine because its secure af (but harder to set up).
* SSHFS: an extension for connecting to a SFTP server on the clients side that lets you mount the server as a network drive (thunar allows this)

### Prevent timeout for SSH/FTP/SFTP sessions
Use client AND server side configuration.\
**Client:** ```echo "ServerAliveInterval 60" >> ~/.ssh/config```.\
**Server:** ```echo "ClientAliveInterval 120\nClientAliveCountMax 720" >> /etc/ssh/sshd_config```.\
Server makes client send 1 null packet every 120s a maximum of 720 times. 120*720=24 hours. 

Heres the copy pasta version for configuration!
```
echo "ServerAliveInterval 60" >> ~/.ssh/config
echo "ClientAliveInterval 120" >> /etc/ssh/sshd_config
echo "ClientAliveCountMax 720" >> /etc/ssh/sshd_config
```

### SSHFS Vs SFTP
* SFTP is a common protocol for graphical ftp clients such as filezilla or network drive mounting with Thunar.
* SSHFS is easier to configure and more lightweight than using virtual file systems inside your graphical file manager. Instead you mound a drive over the command line.
* SSHFS runs over SFTP and therefore you need port 22 open.
* SSHFS also avoids having to create *sftp exclusive* users who are unnable to SSH into the server. for example:
```
Match group sftp
ChrootDirectory /home
X11Forwarding no
AllowTcpForwarding no
ForceCommand internal-sftp <- this command will only allow sftp
```

### Transfer a file with FTP/SFTP
1. Install UFW for an internet condom (though this isnt required)
2. Install a FTP server. vsftpd is good :)
3. Allow only some special local hosts ```/etc/hosts.allow``` should contain ```vsftpd: 192.168.0.0/255.255.255.0```.
4. Set up ```anonymous_enable=NO``` and ```write_enable=YES``` on the host. no config required on the client.
5. Start and stop ```vsftpd.service``` when you need FTP (emphasis on stopping it if you are too lazy to secure it like me)

To transfer files you can initiate ftp or sftp by typing ```ftp destination-ip``` or ```sftp destination-ip```. The prompt will change to show you you are in ftp/sftp mode.
* To Download a file to your machine (from the *destination-ip*) type ```get some-file.txt```.
* To move a file from your machine (to the *destination-ip*) type ```put some-file.txt```.
* To transfer multiple files or folders use ```-r``` and ```/*```. For example ```get /home/roland/folder/* /home/roland/local_location -r```

### Securing VSFTPD (WIP)
will do this later

### SSHFS config
1. Make sure port 22 is open on the server you are connecting to ```sudo ufw status``` and enable port 22 if needed with ```sudo ufw allow 22```. Also make sure you have the packages openssh, sshfs, openssh-server, and fuse.

2. The next thing to do is make sure ```/etc/ssh/sshd_config``` contains the correct path to your sftp server. On my current system (ubuntu 18.04) the rule i have is ```Subsystem       sftp    /usr/lib/openssh/sftp-server```. A hint to finding the location is to use ```whereis sftp-server```.

3. Create a SSH key and copy it to the server to make logging in secure and easier in the future. ```ssh-keygen``` ```ssh-copy-id roland@11.22.33.44```. Then Create an entry in ~/.ssh/config with an alias for the server because sshfs reads the *ssh pub key* from this config to authenticate with the server. 
```
Host TestServer
  HostName 11.22.33.44
  User roland
  Port 22
  IdentityFile /home/roland/.ssh/id_rsa
```

To Mount the remote filesystem on your device run the command ```sshfs -F /home/roland/.ssh/config roland@45.77.236.124:/home/roland mount/```. Add
```-o debug``` to enable debugging information
