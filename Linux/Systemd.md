# Systemd

### Checking systemd

* Control systemd units ```sudo systemctl [stop start restart status] myservice```
* Example: ```sudo systemctl restart ssh```

* See the log of systemd services ```journalctl```
* See the log from this boot ```journalctl -b```
* See the log of a specific unit ```journalctl -u myservice```

### Creating Systemd Service Files

* Systemd unit files are stored in `/etc/systemd/system/*`

Creating a simple systemd unit is almost straight forward.
For example, lets say you wanted to run a script that ran a nodejs script

1. Create a systemd .service file in `/etc/systemd/system/myservice.service`
2. put the following mandatory content in the systemd file

```none
[Unit]
#Describe what the unit does
Description=build my website
#When should the unit run
After=network.target
#
[Service]
#Set the type
Type=oneshot
#set the directory to work from
WorkingDirectory=/home/roland/staticFolio
#Do this before ExecStart
ExecStartPre=/usr/bin/bash -c 'echo build script ran at $(date) >> /home/roland/log.txt'
#Run this command
ExecStart=/home/roland/.nvm/versions/node/v12.16.1/bin/node build.js
User=roland
Group=roland
#
[Install]
WantedBy= multi-user.target
```

#### Simple

If you ran the service on the command line and the service runs forever (until you press control C) the service is simple

#### Forking

If you ran the service on the command line and the service returned but kept running in the background (ie. as a daemon; daemonizing itself) then the service is forking

#### Oneshot

If you ran the service on the command line and it returns without daemonizing itself then the service is oneshot.

* oneshot services benefit from using `ExecStart` and `ExecStop` to set and unset values. If you are doing something like setting a value and then un-setting it once the service is finished running you should also use `RemainAfterExit=true` so that systemd keeps track of the "state" of the service to keep track of which state the service is in

#### Other types

There are other special case types that i haven't learnt about

* type = dbus
* type = notify

### Systemd timers

When you want to run a systemd service file on a regular basis (once every X) you need a corresponding myservice.timer file.

* The myservice.timer should have the same name as myservice.service

A timer might look like this.

```none
[Timer]
#start 10mins after boot
OnBootSec=10 m
#Run once a day at midnight
OnCalendar=daily
#THis is the service that we are targeting
Unit=myservice.service
#
[Install]
WantedBy=timers.target
```

### Types of systemd Service Targets

#### WantedBy=multi-user.target

Established when the system has reached runlevel 2 (a non graphical (gfx) shell where a user is logged in)

#### WantedBy=timers.target

Established when the system has finished setting up all timer units

### Types of Systemd Services

Concerning the `type = simple/forking/oneshot` field, i found some great explanations to what these fields do that i have reworded here for brevity.
