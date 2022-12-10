# SSH

## Designing an SSH Agent Systemd Unit

Running the ssh agent via Systemd is a good idea because you can benefit from its process management
and journaling.

The ssh agent is designed to be run as the user, so we will be using the `systemd --user` flag.

Create the following unit file in `/usr/lib/systemd/user/ssh-agent.service`

```none
[Unit]
Description=OpenSSH Agent
Documentation=man:ssh-agent(1)
Before=graphical-session-pre.target
Wants=dbus.socket
After=dbus.socket

[Service]
Environment=SSH_AUTH_SOCK=%t/ssh-agent.socket
ExecStart=/usr/bin/ssh-agent -D -a %t/ssh-agent.socket
ExecStartPost=/bin/sh -c "echo $MAINPID > /tmp/ssh-agent.pid;"
ExecStop=/bin/sh -c "source export $(cat /tmp/ssh-agent.pid) && /usr/bin/ssh-agent -k"

[Install]
WantedBy=multi-user.target
```

In this case, `%t` refers to the working directory which will usually resolve to `/run/user/1000`.

Lets break down what each line does in the `[Service]`.

First we set the `Environment` variable `SSH_AUTH_SOCK` to `/run/user/1000/ssh-agent.socket`.
This will help the `ExecStart` find the socket when it starts the agent.

Next `ExecStart` starts the ssh agent by calling `-D` to run in the foreground (do not fork),
this makes it a bit nicer for systemd to monitor. And the `-a` flag binds it to the socket.

Once systemd as run `ExecStart` it runs `ExecStartPost` which reads `$MAINPID`
which systemd magically has (hence why `-D` when starting the agent). We write this to a file which
will contain the PID of the ssh-agent process.
This will be useful when we stop the ssh-agent service.

Last we have `ExecStop` which uses `sh` and sources the PID file,
allowing it to shut down the service. Note it also requires `SSH_AUTH_SOCK` to kill the process
gracefully so it can close the socket.

Next we need to glue this in with our shell and tell it where to find the socket.
When commands like `ssh` try to connect to a remote host, it checks the ssh-agent via this socket
to determine if there are any keys in its keyring that it can use for passwordless access.

The below code belongs in your `bashrc` or `zshrc`.

```bash
# check if the ssh-agent.socket file exists in the user runtime directory
if [[ -e "$XDG_RUNTIME_DIR/ssh-agent.socket" ]]; then
  export SSH_AUTH_SOCK="$XDG_RUNTIME_DIR/ssh-agent.socket"
fi
```

The unit can be controlled like so.

```none
# reload the daemon to refresh the files that we just edited
systemctl --user daemon-reload

# enable and start the service
systemctl --user enable --now ssh-agent.service

# get its status
systemctl --user status ssh-agent.service

# stop the service
systemctl --user stop ssh-agent.service

# use journalctl to inspect its logs
journalctl --user -u ssh-agent.service
```

## Display a cool banner when logging in with SSH

Modify */etc/issue.net* and write a cool message in there. Then make sure banners are enabled.

```none
#/etc/ssh/sshd_config
Banner /etc/issue.net
```

## Display a cool login message when logging in with SSH

Switch user to root and modify the bash scripts in `/etc/update-motd.d/`.
The number of the file indicate the order in which they are executed.
