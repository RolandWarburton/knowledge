### Display a cool banner when logging in with SSH
Modify */etc/issue.net* and write a cool message in there. then make sure banners are enabled.
``` 
#/etc/ssh/sshd_config 
Banner /etc/issue.net
```

### Display a cool login message when logging in with SSH
Switch user to root and modify the bash scripts in ```/etc/update-motd.d/```. The number of the file indicate the order in which they are executed.
