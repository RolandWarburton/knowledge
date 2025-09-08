# Bluetooth

## Installing and setting up

Install these pre-reqs for Bluetooth on your desktop.

```bash
sudo apt install xdg-desktop-portal xdg-desktop-portal-wlr xwayland libspa-0.2-bluetooth
```

Ensure the desktop portal is running.

```bash
systemctl --user status xdg-desktop-portal-wlr
```

Ensure these environment variables are defined.
`XDG_CURRENT_DESKTOP` might not be set, if so please set it in your desktop init script.

```bash
echo $XDG_CURRENT_DESKTOP
echo $WAYLAND_DISPLAY

# if XDG_CURRENT_DESKTOP is not set please init in DE startup script
export XDG_CURRENT_DESKTOP=sway
```

Import critical variables into dbus.
Make sure to **put this in your sway config**.

```bash
exec·dbus-update-activation-environment·--systemd·WAYLAND_DISPLAY·XDG_CURRENT_DESKTOP=sway
```

## Using

Restart pipewire and wireplumber

```bash
systemctl --user restart pipewire wireplumber
```

Start scanning for devices.

```bash
bluetoothctl
power on
agent on
default-agent
scan on
```

Then you can connect to your bluetooth device,
it might take a bit of searching as the scan talk can be quite chatty.

```bash
pair AA:BB:CC:DD:EE:FF
trust AA:BB:CC:DD:EE:FF
connect AA:BB:CC:DD:EE:FF
quit
```

It can help to alias this connection step (optional) if you like.

```bash
alias bt-connect='echo -e "connect 14:3F:A6:1C:6B:30\nquit" | bluetoothctl'
```
