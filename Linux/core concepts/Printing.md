# Printing

## Installing CUPS

Linux uses the CUPS printer daemon to communicate to printers. Sometimes you need additional drivers for printers like brother printers that can be found on the AUR.

Install the required software.

```none
sudo pacman -S cups
```

Once you have installed CUPS start its daemon as a service.

```none
sudo systemctl start org.cups.cupsd.service
```

```output
● org.cups.cupsd.service - CUPS Scheduler
     Loaded: loaded (/usr/lib/systemd/system/org.cups.cupsd.service; disabled; vendor preset: disabled)
     Active: active (running) since Wed 2020-06-24 18:09:16 AEST; 47s ago
       Docs: man:cupsd(8)
   Main PID: 256171 (cupsd)
     Status: "Scheduler is running..."
      Tasks: 1 (limit: 19145)
     Memory: 3.7M
     CGroup: /system.slice/org.cups.cupsd.service
             └─256171 /usr/bin/cupsd -l

Jun 24 18:09:16 Arch-Desktop systemd[1]: Starting CUPS Scheduler...
Jun 24 18:09:16 Arch-Desktop systemd[1]: Started CUPS Scheduler.
```

next you can navigate to the web interface of CUPS and add a new printer. go to [http://localhost:631/](http://localhost:631/) in your browser.

When prompted to log in use your computer credentials.

## Debugging

Make sure that your printer is detected in the first place. If you have a brother printer you can grep for Brother to find it on the list of USB devices

```none
lsusb | grep "*Brother*"
```
