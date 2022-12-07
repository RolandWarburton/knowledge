# Sway

## Navigating

### splitting direction

`mod+v` and `mod+b` to `vsplit` and `split` windows.

### Changing Layout

Press `$mod+e` for the standard layout.

```none
+----++----+
|    ||    |
|win1||win2|
|    ||    |
+----++----+
```

Press `$mod+s` for stacking layout.

```none
+----+
|win2| <-- label for next window in stack
+----+
|    |
|win1|
|    |
+----+
```

Press `$mod+w` for tabbed layout.

```none
+---------+
|win1|win2|
+---------+
|         |
|  win1   |
|         |
+---------+
```

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

### Three Window Two Column Layouts

Assume that all window layouts begin from a common starting point.

```none
          root
            | 
            v 
    +-------+-------+
    |       |       |
    v       v       v
  win1    win2    win3

+------------------------+
|+------++------++------+|
||      ||      ||      ||
||      ||      ||      ||
|| win1 || win2 || win3 ||
||      ||      ||      ||
||      ||      ||      ||
|+------++------++------+|
+------------------------+
```

If you have 3 windows stacked in the opposite direction (vertically) you can change their
orientation by pressing `mod+e` again.

```none
+------------------------+       +------------------------+
|+----------------------+|       |+------++------++------+|
||         win1         ||       ||      ||      ||      ||
|+----------------------+|       ||      ||      ||      ||
|+----------------------+|       ||      ||      ||      ||
||         win2         || ----> || win1 || win2 || win3 || 
|+----------------------+|   |   ||      ||      ||      ||
|+----------------------+|   |   ||      ||      ||      ||
||         win3         ||   |   ||      ||      ||      ||
|+----------------------+|   |   |+------++------++------+|
+------------------------+   |   +------------------------+
                             v
                             pressing `mod+e` to swap between vertical and horizontal.
```

Say we want a layout like below.

```none
+------------------------+
|+------++--------------+|
||      ||    win2      ||
|| win1 |+--------------+|
||      |+--------------+|
||      ||    win3      ||
|+------++--------------+|
+------------------------+
```

1. Select any window (in this example window 1 is selected denoted by the `///` shading)
2. First take any window and move it to the top with `mod+k`.
3. Then take window 1 and push it to the left with `mod+h`.

```none
+------------------------+      +------------------------+      +------------------------+
|+------++------++------+|      |+----------------------+|      |+------++--------------+|
|| //// ||      ||      ||      || /////// win1 /////// ||      || //// ||    win2      ||
|| //// ||      ||      ||      |+----------------------+|      || //// |+--------------+|
|| win1 || win2 || win3 || ---> |                        | ---> || win1 |+--------------+|
|| //// ||      ||      ||      |+----------+-----------+|      || //// ||              ||
|| //// ||      ||      ||      ||    win2  |    win3   ||      || //// ||    win3      ||
|+------++------++------+|      |+----------+-----------+|      |+------++--------------+|
+------------------------+      +------------------------+      +------------------------+
|                               |
\ mod+k                         \ mod+h
```

From this point the graph will look like this

```none
  root
    |
    v
  <------+
  |      |
  v      v
win1    con
         |
     +---+---+
     v       v
    win2    win3
```

Whilst on any of the `con` group windows (win1, and win2)
you can change the layout of that container to be in different modes.

Possible modes are `mod+e`, `mod+w`, and `mod+s`. The default is `e`
in which you can see all 3 windows at once.

The **stacking** view can be activated with `mod+s` and looks like this.


```none
+------------------------+
|+------++--------------+|
||      ||    win2      || <- windows are stacked up here as banners
||      |+--------------+|
|| win1 ||              ||
||      ||              ||
||      ||    win3      ||
|+------++--------------+|
+------------------------+
```

The **tabbed** view can be activated with `mod+w` and looks like this.

```none
+------------------------+
|+------++--------------+|
||      ||  win2 | win3 || <- windows are tabbed up here as banners
||      |+--------------+|
|| win1 ||              ||
||      ||              ||
||      ||    win2      ||
|+------++--------------+|
+------------------------+
```

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

## Searching The Sway Tree Programmatically

Find the currently active window.

```bash
swaymsg -t get_tree | jq '.. | select(type == "object") | select(.focused == true)'
```

Get a filtered list of windows.

```bash
swaymsg -t get_tree | \
jq '.. | select(.name?) |
{
  name: .name,
  visible: .true,
  type: .type,
  id: .id,
  contains_nodes: (has("nodes") | tostring)
}'
```

## Sway Config

### Sway Config Variables

`@sysconfdir@` is typically found in `/usr/local/etc/sway/`.

The `@sysconfdir@` variable represents the location of the system-wide configuration directory.

`@datadir@` is typically found in `/usr/share/sway`

The `@datadir@` variable in Wayland Sway refers to the directory where Sway stores its data files,
such as configuration files, themes, and icons.
