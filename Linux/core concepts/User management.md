# User Management

### All user accounts

```none
ls /etc/psswd
```

### Add a user

* `sudo adduser roland`
* `sudo usermod -aG sudo roland` -aG **A**ppend the '*sudo*' **G**roup to the user.

### Change a users shell

See avaliable shells with `cat /etc/shells` \
Change the shell with `sudo usermod --shell /bin/bash`

### Make a user directory after the user is created

Make suro to run as root or sudo. `sudo mkhomedir_helper roland`

### List all users

```none
cat /etc/passwd
```

### List all groups

```none
cat /etc/group
```

### See the groups a user is in

`groups USERNAME` or `groups` for to display the current user

### Create a new group

`sudo addgroup GROUPNAME`. the groupname can be anything. For example an sftp group could be `sudo addgroup sftp`

### Add a new group to a user

```none
su -
usermod -G sudo,sftp,anothergroup roland
````

Or append that user to the sudo group

```none
usermod -aG sudo roland
```
