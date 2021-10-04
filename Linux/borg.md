# Borg Backup

Finally, time to get a better backup solution than running rsync on a cron job.

## Installing

Debian 11 is shipping a new enough version of borg (1.1.6).

```none
sudo apt install borgbackup
```

## Understanding how borg works

### Borg Parts

To create backups you should understand these concepts

#### Borg Repositories

A borg repository is where everything (including actual backups) are stored.

#### Borg Archive

A borg archive is a single backup. A borg archive is placed inside the borg repository.

You can find encrypted archives at `/path/to/repo/data`.

### Quick Start

Create a borg **repository**.

```none
borg init --encryption=repokey /home/roland/.borgrepo
```

Create an **archive** called test in the **repository** that backups up the folder `~/backmeup`.

```none
borg create .borgrepo::test ~/backmeup
```

To restore the archive use the following command from the **root** of the file system.

```none
cd /
borg extract /home/roland/.borgrepo::test
```

## Extra Command Line Options

When doing common tasks like archiving and restoring you can use these flags.

```none
borg create --stats ~/.borgrepo::test3 ~/backmeup

borg create --list ~/.borgrepo::test2 ~/backmeup

borg create --progress ~/.borgrepo::test3 ~/backmeup

borg create --verbose ~/.borgrepo::test4 ~/backmeup

borg create --info ~/.borgrepo::test5 ~/backmeup
```

## Running Borg in Scripts

Just export `BORG_PASSPHRASE` environment variable.

```none
export BORG_PASSPHRASE=password123
```

## Notes on Space Requirements

Borg docs recomends that you ensure that you ALWAYS have enough free space on your backup device.

To monitor the space that borg is using for all your archives in a repository you can use the following command.

```none
borg info ~/.borgrepo::archive
```

Another config option you can use is to change the additional_free_space config option in `/path/to/repo/config`.

```none
borg config /path/to/repo additional_free_space 5G
```

So there are a few things you can engineer to ensure that you have enough space.

### Use LVM to Allow for a Buffer

Create a LVM and set its size to ~90% so if you run out of space you can resize and have enough space to run borg.

### Writing a script to extract the size of the last archive

First extract the name of the most recent archive. Make sure you have `jq` installed.

```none
borg list ~/.borgrepo --last 1 --json | jq ".archives[0].name"
```

```output
test6
```

The other component we need is to get the size of the archive.

```none
borg info ~/.borgrepo::test6 --json | jq ".archives[0].stats.original_size"
```

This will output in bytes

```output
24
```

So given this information we can write a script that contains some logic to check the remaining space on the backup drive and compare it to the size of the last archive plus a buffer to see if we probably have enough space. The use of `-r` in jq is to get the raw output and remove the double quotes.

```bash
LASTARCHIVE=$(borg list ~/.borgrepo --last 1 --json | jq -r ".archives[0].name")
LASTARCHIVESIZE=$(borg info ~/.borgrepo::$LASTARCHIVE --json | jq -r ".archives[0].stats.original_size")
echo "Last archive size is $LASTARCHIVESIZE"
```

## Remote Borg Server

### Configure Remote Borg Server and Client

#### Step 1. (Client) Generate SSH Key

Generate your ssh key.

```none
ssh-keygen -f ~/.ssh/id_borg
```

Then configure on the client to use this key when connecting to borg@store.rolandw.lan.

```none
# /home/roland/.ssh/config

Host borg@store.rolandw.lan
    IdentityFIle /home/roland/.ssh/id_borg
```

#### Step 2. (Server) Add SSH Key to authorized_keys

Manually install your SSH key on the remote server, this is because we need to add some restrictions to users logging in with this key.

Get the public key from the client first. Then add it to the authorized_keys file on the remote server with the prepended command.

I found that in some tutorials the use of the `command="/usr/local/bin/borg serve --restrict-to-path /volume2/borgbackup",restrict` command did not work for me, so instead of resorted to using just the `restrict` command, see below.

```none
# store.rolandw.lan
# /home/borg/.ssh/authorized_keys

restrict sh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCneUnNyOfUoTY3aE1cGHRIwwlZDOB2k2Q2pd8tNKcbufCSeysTmlvYuVbKWPUp4qtcsMTbDO7IDT6JPwJuvxHjvP1x2eyALg/1p5+xLMFapoKKQCtiZKZv753WsjMSF9ycrdyvhYz1dM5nYmBlpK7PRJckzFqLfv4BXJx7myen/aO11scaId/r7LAQh570ULnI6i/aziRxHpdUGRF2dvlvO2bp43yqrcKc6WXThJ8pMnCkQ0OrEFertXjUdpap+FMECXWzNQY9iClRRA/6VXzoAc2sKfLQzqgSrz36VbYyupRRmZMIg81l6qBYhKhNKJ5JQxrg/WfT82veQX4m15yo6BpHRJ3iiHLVKX7e2x5QfYfXUWSiUVUsp1Olvm4DT9AeodgQLuCdfGO2U1bYusnNSutRoC4G1RU5uWr5E7mUDLRESszEZ81dlX9xj6m2/Huoat+PLmamTyEnMbnb4rofNM7ZioSxyiuOmseKWIVrPkflFoPQFDzziG1ZnxPRYIM= roland@debian
```

#### Step3. (Client) Init Borg as a Client

Create a new repo called desktop on the client.

```none
# init the backup repo from the client
borg init --encryption=repokey borg@store.rolandw.lan:desktop

# get a backup of the key just in case. put it somewhere safe (do not share it)
borg key export borg@store.rolandw.lan:desktop ~/borg-desktop-1-backup.key
```

After this is run you can see the borg server added to the client. by checking `~/.config/borg`.

## Configure Backup Scripts on the Client

### Example Command

**Client** - Create scripts for backing up to the remote borg server.

Creating backup tasks, from the client run these commands on a cron or systemd timer.

```bash
# for brevity in scripts
export BORG_REPO=borg@store.rolandw.lan:desktop
export BORG_PASSPHRASE=password123
export BORG_RSH='ssh -i /home/roland/.ssh/id_borg'

borg create \
  --verbose --filter AME \
  --list --stats --show-rc \
  --exclude-caches \
  ::'{hostname}-daily-{now}' \
  /home/roland
```

I took this command from [here](https://practical-admin.com/blog/backups-using-borg/), the author also provides explanation for what each flag does.

```output
# we want borg to create a new backup
borg create
 
# output verbose logging
  --verbose
 
# only backup new, updated/modified, or files which couldn't previously be backed up
  --filter AME
 
# output all files considered for backup, even if no action was taken
  --list
 
# show statistics about the backup when done
  --stats
 
# print the return code to the output
  --show-rc

# exclude any directory with a CACHEDIR.TAG file
  --exclude-caches

# this line is simplified because we exported the borg repo name into an environment variable
  ::'{hostname}-monthly-{now}
```

Filtering flags `--filter AME` meaning:

More flags [here](https://borgbackup.readthedocs.io/en/stable/usage/create.html#item-flags).

* **A** - regular file, added (see also I am seeing "A" (added) status for an unchanged file!? in the FAQ)
* **M** - regular file, modified
* **U** - regular file, unchanged
* **E** - regular file, an error happened while accessing/reading this file

Note for "A" Added when file has already been added to the archive.

> The files cache is used to determine whether Borg already "knows" / has backed up a file and if so, to skip the file from chunking. It intentionally excludes files that have a timestamp which is the same as the newest timestamp in the created archive. So, if you see an "A" status for unchanged file(s), they are likely the files with the most recent timestamp in that archive.

#### Working Script Example

Create the script to run on the client which runs some borg tasks.

```bash
#!/bin/bash

#  /usr/local/bin/borg-daily.sh
export BORG_REPO=borg@store.rolandw.lan:desktop
export BORG_PASSPHRASE=Password123
export BORG_RSH='ssh -i /home/roland/.ssh/id_borg'

# some helpers and error handling:
info() { printf "\n%s %s\n\n" "$(date)" "$*" >&2; }
trap 'echo $( date ) Backup interrupted >&2; exit 2' INT TERM

info "Starting backup"

borg create \
    --verbose --filter E \
    --list --stats --show-rc \
    --exclude-caches \
    --exclude '/home/*/.cache/*' \
    --exclude /home/roland/.local/share/Trash \
    borg@store.rolandw.lan:desktop::'{hostname}-daily-{now}' \
    /home/roland

backup_exit=$?

info "Pruning repository"

# After the backup is done we can prune old archives

# prune the repo
borg prune \
    --list \
    --prefix '{hostname}-daily-' \
    --show-rc \
    --keep-daily 7 \
    --keep-weekly 4 \
    --keep-monthly 3 2>&1

prune_exit=$?

# use highest exit code as exit code
global_exit=$((backup_exit > prune_exit ? backup_exit : prune_exit))

if [ ${global_exit} -eq 1 ]; then
    info "Backup and/or Prune finished with a warning"
fi

if [ ${global_exit} -gt 1 ]; then
    info "Backup and/or Prune finished with an error"
fi

exit ${global_exit}
```

To run automatically on a daily basis you can use systemd timers and unit files on the client.

Place the following in the `/etc/systemd/system/borg-backup.service` file.

```none
[Unit]
Description=Borg Backup

[Service]
# To avoid permission errors with the backup use the current user/group
User=roland
Group=roland

Type=simple

# We do not want to interrupt other important tasks so we set the scheduling parameters accordingly
Nice=19
IOSchedulingClass=2
IOSchedulingPriority=7

# run it!
ExecStart=/usr/local/bin/borg-daily.sh

[Install]
WantedBy=multi-user.target
```

These following values exist because they were in the example script.

* The IOSchedulingPriority is a value between 0 and 7 where 0 is the highest priority.
* The IOSchedulingClass is a value between 0 and 3

Place the following timer in the `/etc/systemd/system/borg-backup.timer` file.

```none
[Unit]
Description=Borg Backup Timer

[Timer]
WakeSystem=false
# this will trigger the unit at 3am every day
OnCalendar=*-*-* 03:00

[Install]
WantedBy=timers.target
```

Then reload systemd `sudo systemctl daemon-reload` and enable the unit `sudo systemctl enable --now borg-backup`

To monitor the backup jobs run `sudo journalctl -u borg-backup.timer`.
