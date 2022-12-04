# Sway

## Navigating

### splitting direction

`mod+v` and `mod+b` to `vsplit` and `split` windows.

### resizing

`mod+r` or `mod+lmouse`.

### pop window out

### mod+shift+space to pop a window into floating mode

`mod+space` to swap focus between tiling windows and floating windows.

### window tiling direction

To convert two side-by-side windows `| | |` into a top-to-bottom `|-|`
use mod+e. This will toggle the direction of every window to be opposite of what it currently is.

### moving windows

`mod+shift+arrow` will move the window.

## Controlling Sway Remotely

### Using WAYLAND_DISPLAY (swaynag example)

Sway exposes some environment variables. Mainly `WAYLAND_DISPLAY` for the purposes of targeting
sway sessions that are currently active.

To run the `swaynag` command to send a message to an active sway session you can run the following.

```none
ssh user@host "WAYLAND_DISPLAY=wayland-1 swaynag -m 'dinner is ready'"
```

The `WAYLAND_DISPLAY` can be obtained by running `echo $WAYLAND_DISPLAY`
on the computer running wayland.

### Using sway-ipc (swaymsg example)

Swaymsg is a command line utility that comes with sway that allows you
to control it programmatically.

Here's some examples of using swaymsg to control the active work space in sway.

By default, the trick above where we called `swaynag` won't work here.
To control this remotely we need to do a couple things.

When we run swaymsg we need to point to the IPC socket which is usually
`/run/user/1000/sway/ipc*.sock`. This file changes name, for example.

```none
/run/user/1000/sway-ipc.1000.245075.sock
```

The `245075` number refers to the process ID of the Sway compositor.
The `1000` number refers to the user ID of the user who is running the Sway compositor.

While SSH inside the wayland machine. This command should work.

This file location can be obtained by inspecting `$SWAYSOCK` on the sway workstation.

```none
swaymsg -s /run/user/1000/sway-ipc.1000.245075.sock -t get_workspaces
```

## Sway Config

### Sway Config Variables

`@sysconfdir@` is typically found in `/usr/local/etc/sway/`.

The `@sysconfdir@` variable represents the location of the system-wide configuration directory.

`@datadir@` is typically found in `/usr/share/sway`

The `@datadir@` variable in Wayland Sway refers to the directory where Sway stores its data files,
such as configuration files, themes, and icons.
