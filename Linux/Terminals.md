### Set default Lines and Columns
Export the ```$Lines``` and ```$COLUMNS``` variables to set it for terminals who respect them (all of them more or less).
The default on my computer was 50 lines * 112 columns

Terminals that respect ```~/.Xresources``` can also be configured via ```TerminalName*geometry:  240x84```. 
The defult for Xresources is 80 lines * 24 columns.

### Terminal key codes
Its useful to know the keycodes when you are mapping buttons to do things.

```showkey -a```
Example:
```
a 	97 0141 0x61
b 	98 0142 0x62
c 	99 0143 0x63
1 	49 0061 0x31
2 	50 0062 0x32
3 	51 0063 0x33
```

##### TermInfo files: 
```ls /usr/share/terminfo/x```\
List of supported terminals. use ```echo $TERM``` to what terminal the system thinks its using.\
You can export a new $TERM from the list above.
