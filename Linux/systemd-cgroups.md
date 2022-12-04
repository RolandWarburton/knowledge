# Systemd Control Groups

See all the systemd managed processes in the user slice.

```none
systemd-cgls -u user.slice
```

A `htop` like output of slice resource utilization.

```none
systemd-cgtop
```

Inspect a specific slice.

```none
systemd-cgtop --group /user.slice
```

## Group Samples

The following sample is taken from my laptop running a TTY.

The result of `systemd-cgls -u user.slice`.

```none
Unit user.slice (/user.slice):
└─user-1000.slice
  ├─user@1000.service
  │ ├─session.slice
  │ │ ├─pipewire-pulse.service
  │ │ │ └─14586 /usr/bin/pipewire-pulse
  │ │ ├─wireplumber.service
  │ │ │ └─14585 /usr/bin/wireplumber
  │ │ └─pipewire.service
  │ │   └─14584 /usr/bin/pipewire
  │ ├─app.slice
  │ │ └─dbus.service
  │ │   └─795 /usr/bin/dbus-daemon --session --address=systemd: --nofork --nopi…
  │ └─init.scope
  │   ├─652 /lib/systemd/systemd --user
  │   └─653 (sd-pam)
  └─session-1.scope
    ├─   536 /bin/login -p --
    ├─   667 -zsh
    └─275418 systemd-cgls -u user.slice
```

Notice the session-1 contains the TTY.

Next lets start up the desktop (sway over wayland).

```none
dbus-run-session sway
```

```none
Unit user.slice (/user.slice):
└─user-1000.slice
  ├─user@1000.service
  │ ├─session.slice
  │ │ ├─pipewire-pulse.service
  │ │ │ └─14586 /usr/bin/pipewire-pulse
  │ │ ├─wireplumber.service
  │ │ │ └─14585 /usr/bin/wireplumber
  │ │ └─pipewire.service
  │ │   └─14584 /usr/bin/pipewire
  │ ├─app.slice
  │ │ └─dbus.service
  │ │   └─795 /usr/bin/dbus-daemon --session --address=systemd: --nofork --nopi…
  │ └─init.scope
  │   ├─652 /lib/systemd/systemd --user
  │   └─653 (sd-pam)
  └─session-1.scope
    ├─   536 /bin/login -p --
    ├─   667 -zsh
    ├─275529 dbus-run-session sway
    ├─275530 dbus-daemon --nofork --print-address 4 --session
    ├─275531 sway
    ├─275543 swaybar -b bar-0
    ├─275545 sh -c wl-paste -t text --watch clipman store
    ├─275546 wl-paste -t text --watch clipman store
    ├─275547 sh -c while date +'%Y-%m-%d %I:%M:%S %p'; do sleep 1; done
    ├─275555 sh -c foot
    ├─275556 foot
    ├─275557 /usr/bin/zsh
    ├─275963 sleep 1
    └─275964 systemd-cgls -u user.slice
```

We still have just one session `session-1`, but it now contains more things.
Mainly our sway process and its associated software (swaybar, foot).

The use of the `dbus-run-session` command has also resulted in 
`dbus-daemon·--nofork·--print-address·4·--session` existing in `session-1`.

The dbus-daemon flags are described below:

* When used together, `dbus-run-session` will start a D-Bus session and then run the `dbus-daemon`
command to start the daemon that connects to the dbus session.
* The --nofork option tells the dbus-daemon command not to fork itself into the background,
so that it runs in the foreground and sends its output to the console.
* The --print-address option tells the dbus-daemon command to print the address
of the D-Bus message bus to the console. 
* The 4 argument specifies the bus address family, which can be either 0 (AF_UNIX) or 4 (AF_INET),
depending on whether the D-Bus message bus is using a UNIX domain socket or an Internet socket.
* The --session option tells the dbus-daemon command to start a session bus,
which is a private D-Bus message bus that is used by the user's desktop session.

Lets now ssh into the computer and observe a new session scope being created.

```none
Unit user.slice (/user.slice):
└─user-1000.slice
  ├─user@1000.service
  │ ├─session.slice
  │ │ ├─pipewire-pulse.service
  │ │ │ └─14586 /usr/bin/pipewire-pulse
  │ │ ├─wireplumber.service
  │ │ │ └─14585 /usr/bin/wireplumber
  │ │ └─pipewire.service
  │ │   └─14584 /usr/bin/pipewire
  │ ├─app.slice
  │ │ └─dbus.service
  │ │   └─795 /usr/bin/dbus-daemon --session --address=systemd: --nofork --nopidfile --systemd-activation --syslog-only
  │ └─init.scope
  │   ├─652 /lib/systemd/systemd --user
  │   └─653 (sd-pam)
  ├─session-85.scope
  │ ├─275980 sshd: roland [priv]
  │ ├─275986 sshd: roland@pts/0
  │ ├─275989 -zsh
  │ ├─276007 ssh-agent -s
  │ ├─279782 systemd-cgls -u user.slice
  │ └─279783 pager
  └─session-1.scope
    ├─   536 /bin/login -p --
    ├─   667 -zsh
    ├─275529 dbus-run-session sway
    ├─275530 dbus-daemon --nofork --print-address 4 --session
    ├─275531 sway
    ...
    └─279778 sleep 1
```

As you can see `session-85` is now created running zsh.
