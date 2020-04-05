# Networking (Part Three). SSH stuff
### Hardening SSH access (WIP)
limit connection attempts per minute ```sudo apt-get install ufw && sudo ufw limit OpenSSH```

### Log in with no password with SSH
1. Generate a new rsa key with ```ssh-keygen```. Do not specify a password when asked.
2. Add this key to the list of keys that ssh will ask when authenticating ```ssh-add ~/.ssh/id_nopass_rsa```. This list is a global list of possible ssh keys that your client will try against the server.
3. Give correct file permissions. **700 on .ssh** and **640 on authorized_keys**.

### Debugging SSH
#### Connection closed by 45.77.236.124 port 22
check the system time which is one of the MANY causes for this problem. (see *My Time and Date are all fucked up*)

#### Connection hangs on - 'Connecting to a.b.c.d [a.b.c.d] port 22'
check the system time which is one of the MANY causes for this problem. (see *My Time and Date are all fucked up*)
