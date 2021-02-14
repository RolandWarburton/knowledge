# Firefox on Debian

## Uninstall existing versions of FF

```none
sudo apt purge firefox-esr
```

Go to [https://www.mozilla.org/en-US/firefox/new/](https://www.mozilla.org/en-US/firefox/new/) and download the `.tar.bz2`.

Extract the files.

```none
tar -xvf firefox.tar.bz2
```

This will create a firefox directory which needs to be placed in `/opt`.

```none
mv firefox /opt
```

Next you need to create a desktop entry so that programs like application launchers can identify it.

```none
vim /usr/share/applications/firefox.desktop
```

```none
[Desktop Entry]
Name=Firefox
Comment=Web Browser
Exec=/opt/firefox/firefox %u
Terminal=false
Type=Application
Icon=/opt/firefox/browser/chrome/icons/default/default128.png
Categories=Network;WebBrowser;
MimeType=text/html;text/xml;application/xhtml+xml;application/xml;application/vnd.mozilla.xul+xml;application/rss+xml;application/rdf+xml;image/gif;image/jpeg;image/png;x-scheme-handler/http;x-scheme-handler/https;
StartupNotify=true
```

Then if you want to start firefox from the CLI, symlink it to `/usr/local/bin` which is where all unprivileged user programs belong (IE programs you dont run as root).

```none
sudo ln -s /opt/firefox/firefox /usr/local/bin/firefox
```

## TLDR installation script

If you have done this before then just use this script to install firefox in one copy paste!

```bash
if [! -d "/opt/firefox" ]; then
		mkdir -p /tmp/firefox # create a temp location to store the downloaded binary
		wget -O /tmp/firefox/FirefoxSetup.tar.bz2 "https://download.mozilla.org/?product=firefox-latest&os=linux64&lang=en-US"
		sudo tar -xf /tmp/firefox/FirefoxSetup.tar.bz2 --directory /opt # extract firefox to /opt
		sudo cp applications/firefox.desktop /usr/share/applications # copy the desktop shortcut over for app launchers (like rofi) to read
		sudo ln -s /opt/firefox/firefox /usr/local/bin/firefox # symlink the executable to the bin for CLI launching
		sudo rm -rf /tmp/firefox
fi
```

