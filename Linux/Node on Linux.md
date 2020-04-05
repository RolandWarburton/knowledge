# Node on Linux
I want to add more notes here in future. mainly researching and explaining how the nvm install script works (and create separate notes on wget and curl).

Curl reference [here](https://curl.haxx.se/docs/httpscripting.html).

### Installing
Install nodejs, npm, and nvm.
```
sudo pacman -S nodejs npm &&
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash
```

You can then start a normal project with `npm init -y` and work from there.

### How curl works to download nvm
```Curl -o- url | bash ``` (-o = output to file allows the script to write something to the disk). Then pipe it and execute the script from the url in into bash. I do not know what the 2nd - means after o.


### Managing node modules
node modules are located at: ```/usr/local/lib/node_modules```
