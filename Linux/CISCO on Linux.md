# Connect to CISCO equipment through a terminal
To connect to CISCO equipment through serial you can use minicom.
To run minicom for CISCO equipment you need to configure your settings with ```minicom -s```
1. Ensure you are using the correct serial device.
List your devices with ```ls /dev``` and look for the correct device to use to communicate with. In my case my serial cables are rj45 -> usb. So i used /dev/ttyUSB0* as my serial device.
2. Have the correct serial speed. And
Turn Hardware Flow Control off in minicoms settings.

To save the config use the *Save setup as dfl* option. dfl = default.

Connect the the device by running ```minicom```. quit the connection with ```ctrl+a, q```.