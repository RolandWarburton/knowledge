# Automating Photo Backups

While its nice to rely on google photos to manage all of my photos. It would be nice to have a local backup of my photos taken on my phone just in cast something goes wrong with google photos, IE i run out of space or my account is deactivated.

To achieve this i use an app for my phone called [FolderSync](https://play.google.com/store/apps/details?id=dk.tacit.android.foldersync.lite&hl=en_AU&gl=US) that lets me use SFTP along with many other possible file transfer methods to back up my photos, including to other cloud storage providers.

I chose to use SFTP over my local network to transfer the contents of `/storage/emulated/0/DCIM/Camera` to `/home/roland/mobile_photos` on my server which has a lot more than the free 15GB that google offers, therefore allowing me to take and store a LOT more photos than i would otherwise be able to on the free google storage tier.

Briefly this is the instructions and settings i used when setting up FolderSync for you (or me in the future) to follow along with. Foldersync works on "profiles" and "folder pairs".

## Setting up folder sync

### Create account

1. Create a SFTP Account under the "Accounts tab"
   1. Login name: the name you use to login to your server over sftp
   2. Password: the password you use to login to your server over sftp
   3. Server address: the IP address of your server (i haven't) checked if domain names work
   4. Port: the ssh port (22)
   5. Path: I left this blank
   6. Known hosts file: I left this blank
   7. Private key file: I generated a new SSH key on the server and uploaded it to my phone as `note9_rsa` (upload the private key, IE not the one ending in .pub)
   8. Key file password: I left this blank because i didn't put a password on the SSH keys
   9. SSH host key fingerprint: When i generated the key on the server it gave me this information and i just copied it onto the phone
   10. Disable compresstion: I left this enabled
   11. Charset: Left to default

### Create folder pair

1. Next create a folder pair using the SFTP account you created
    1. Sync type: To remove folder (one way only)
    2. Use the folder icon buttons on the left to set the local and remote folder
        1. Remote: `/home/roland/mobile_photos`
        2. Local: `/storage/emulated/0/DCIM/Camera`
    3. Scheduling: Daily
    4. Sync subfolders: enabled
    5. Sync hidden files: enabled
    6. Advanced: Enabled
        1. Enable `"Only resync source files if modified (ignore target directory)"` if you plan on moving the files around on the server once they are copied. Otherwise if you move a file it will get re-synced again and you'll end up with duplicate files
        2. Use temp file scheme: enabled
        3. I left everything else disabled

Now when we click the sync button all our photos should upload to the server.

## Organising the photos on the server

To organise the photos on the server we won't do anything fancy. Theres no need to even install a fancy script or anything. Just run this command provided by the exiftool package to sort the photos into year and month folders

```none
exiftool -d %Y/%m "-directory<datetimeoriginal" *.jpg
```

To sort videos (mp4) thats a bit harder and does require a small script to set up. Luckily with the power of `ffprobe` and some python it can be done succinctly. Also be aware that there is better ways of doing this script, just that i am inexperienced at python and throwing together something quickly that does the job (albeit slowly and inefficiently) is better than nothing at the end of the day.

If you care about performance, you could swap ffmpeg with a python library instead to determine the creation date, but my method works for what i need.

```python
from os import popen
from datetime import datetime
from dateutil import parser
import errno
from shutil import move
from os import listdir, mkdir
from os.path import isfile, join, splitext, realpath, dirname


# requires exiftool library from your package manager
def getDestFilepath(filepath):
    # command returns a json string
    # https://stackoverflow.com/questions/31541489/ffmpeg-get-creation-and-or-modification-date
    command = f"ffprobe -v quiet -select_streams v:0 -show_entries stream_tags=creation_time -of default=noprint_wrappers=1:nokey=1 {filepath}"
    # run the command
    stream = popen(command)
    output = stream.readlines()
    # check there was some data returned
    if not len(output) == 1:
        return datetime.today().strftime('%Y-%m-%d')
    else:
        ts = parser.parse(output[0])
        year = ts.year
        month = f"{ts.month:02d}"
        # return the filepath that this mp4 should belong in
        fp = realpath(f"{base_directory}/{year}/{month}")
        return fp


# directory to read
# CHANGE THIS
base_directory = "/home/roland/mobile_photos"

# store files we find here
mp4Files = []

# read every file in the directory
files = [f for f in listdir(base_directory) if isfile(join(base_directory, f))]

# sort .mp4 into an array
for file in files:
    fileParts = splitext(file)
    fileExt = fileParts[1]
    if fileExt == ".mp4":
        mp4Files.append(file)

for file in mp4Files:
    sourceFilepath = realpath(f"{base_directory}/{file}")
    destFilepath = getDestFilepath(sourceFilepath)
    try:
        mkdir(dirname(destFilepath))
    except OSError as exc:
        if exc.errno != errno.EEXIST:
            raise
        pass
    # then try to move the file
    try:
        # if the file already exists the move command will skip it
        move(sourceFilepath, destFilepath)
        print(destFilepath)
    except OSError as exc:
        if exc.errno != errno.EEXIST:
            raise
        pass
```

Now that we can organise our photos and mp4s by year and month without creating any major hiccups, we need to automate their deployment.

Lets create some cronjobs for that!

Create files to store our script calls in.

```none
touch ~/bin/processDcimJpgs.sh
touch ~/bin/processDcimMp4s.py

chmod 755 ~/bin/processDcimJpgs.sh
chmod 755 ~/bin/processDcimMp4s.py
```

```shell
# ~/bin/processDcimJpgs.sh

DIR="/home/roland/mobile_photos"
cd $DIR && \
exiftool -d %Y/%m "-directory<datetimeoriginal" *.jpg
```

And copy our python script for MP4s into ~bin as well.

```shell
cp thePyScriptAbove.py ~/bin/processDcimMp4s
```

Next add some stuff to our crontab file using `crontab -e`.

```none
# every day at 1am
0 1 * * * /usr/bin/bash /home/roland/bin/processDcimJpgs.sh
# every day at 2am
0 2 * * * /usr/bin/python3 /home/roland/bin/processDcimMp4s.py
```

You may check the log status of cron through journalctl.

```none
# check all in current rotation
journalctl -u cron.service -a

# check since last boot
journalctl -u cron.service -b

# since the last couple days
journalctl -u cron.service --since "48hour ago"
```
