# Virtualization

So far i havent tried out much virtulization on linux, i plan on experiemnting with esxi 6.5 when i get some hardware that can run it however 🙂.

## VMware

### Fix internet issues

Make sure you have started the networking services.

```none
sudo systemctl start vmware-networks.service
sudo systemctl enable vmware-networks.service
```

Then "redetect" your network devices

```none
sudo modprobe vmnet && sudo vmware-networks --start
```
