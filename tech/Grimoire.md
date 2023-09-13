# Grimoire

Summoning daemons to do our bidding, making lifeless things appear alive,
writing esoteric commands in hermetic languages and tricking rocks into thinking.
Yep, computers are pretty much sorcery.

From [Fred Bednarski](https://fdisk.space/grimoire/),
a grimoire is a collection of useful software bits.

> The grimoire is a scrapbook for useful bits of code,
> software solutions and all kinds of tech wizardry.
> The entries here are something between a reference and a tutorial.

## Disclaimer

Nothing here is guaranteed to work, or even be useful in the long term.
Just a collection of computer "spells" that i have used more than once.

### Convert PNG to PDF

This also applies to any other conversions that imagemagick can do.
The trick is to use basename to avoid `.png.pdf`.

```none
fdfind --max-depth=1 '.png$' --exec sh -c 'convert "$0" "$(basename "$0" .png).pdf"'
```

### Trim Whitespace

You can trim (or slurp) whitespace using the `tr` command which translates or deletes characters.

```none
tr -d '[:space:]'
```

### Setting Brightness

`sysfs` exposes some controls to edit the screen brightness.

Setting the brightness level (eg 600) `echo "600" | sudo tee /sys/class/backlight/*/brightness `).

Getting the maximum brightness value (minimum is 0) `cat /sys/class/backlight/*/max_brightness`.

### Convert PNGs to Squares

You can pad a picture with transparent space to make it a square,
this is useful for websites that require a square profile picture for example.

```none
convert INPUT.png \
-background none \
-gravity center \
-resize 400x400 \
-extent 400x400 \
OUTPUT.png
```

You may update the background color by specifying `-background "#FF0000"`.

The `resize` tag changes the size of the image.
The `extent` tag changes the size of the canvas.

The next section is about perfect resizing.

If you would like resize and extent to have values such that the
minimum amount of transparent border is added you can use below.

```bash
# return the biggest x or y value from the input image dimensions
LENGTH=$(identify -format "%wx%h" INPUT.png | grep -oE '[0-9]+' | sort -nr | head -n 1)

# run the same command again with substituted values
convert INPUT.png \
-background none \
-gravity center \
-resize "$LENGTH"x"$LENGTH" \
-extent "$LENGTH"x"$LENGTH" \
OUTPUT.png
```

### Convert PNGs to 1920x1080 Backgrounds

Take a cool image `input.png` that i want to convert into a desktop background,
i can use this comment to place it on a white 1920x1080 image.

```none
convert input.png \
-resize "1920x1080>" \
-gravity center \
-background white \
-extent 1920x1080 \
output.png
```

`-resize "1920x1080>"` resizes to 1920x1080, it will maintain its aspect ratio.
The > symbol means "only if it's larger."

`-extent 1920x1080` sets the overall size of the image, causing the output to be 1920x1080.

### ESM Get Directory Name

In commonjs (cjs) you used to have `__dirname` as a global variable
which referred to the modules base directory.

When using ESM you can emulate this behavior with this snippet.

```js
import { resolve, dirname } from 'path';

const __dirname = dirname(new URL(import.meta.url).pathname);
```

### Golang Programatic Breakpoint

Insert this line anywhere in your code.

```none
runtime.Breakpoint()
```

### Sway Find Mice And Keyboards

To find mice and return their acceleration speed.

```none
swaymsg -t get_inputs | jq '.[] | select(.type == "pointer") | {name: .name, accel_speed: .libinput.accel_speed}'
```

To find mice with a particular name.

```none
swaymsg -t get_inputs | jq '.[] | select(.type == "pointer" and .name == "Logitech MX Master 3")'
```

You can change the properties of the mouse using the sway `config` file.

```none
input "1133:16514:Logitech_MX_Master_3" {
  pointer_accel -0.6
  accel_profile "flat"
}
```

To find keyboards change the type to "keyboard".

```none
swaymsg -t get_inputs | jq '.[] | select(.type == "keyboard")'
```

### Ignoring Files When FS Searching

When i am searching for things within the file system, i rely on `fdfind` and `ripgrep`.

The shorthand for these tools are `fd` and `rg`.

A lot of the time it helps to exclude directories.

When searching for needles in haystacks, i find myself searching the entire file system
and so must exclude special directories. I find the below to work well so far.

```none
rg 'SEARCH TERM' -g '!sys/' -g '!var/log/auth.log' -g '!run/' -g '!proc/'
```

When i am searching for file names, i can do a similar directory ignore pattern.
Its also worth searching for `--hidden` to include hidden files and folders.

```none
fd --hidden 'SEARCH TERM' --exclude='/sys' --exclude='/run'
```

### Clean up Git Checkout

```bash
# reset tracked files
git reset --hard

# remove un-tracked files
git clean -f
```

### Sed Change Line N Lines After Match

I had a patch i need to apply where i change a "no" to a "yes". The problem is i cant just `sed`
that specific line out because it occurs multiple times in the file.

Below is a diff of the desired change.

```xml
<message>Authentication is required to grant an application high priority scheduling</message>
<message xml:lang="tr">Sürecin yüksek öncelikli çalıştırılabilmesi için yetki gerekiyor</message>
<defaults>
-  <allow_any>no</allow_any>
+  <allow_any>yes</allow_any>
  <allow_inactive>yes</allow_inactive>
  <allow_active>yes</allow_active>
</defaults>
```

No problem with sed.

```none
sed -E '/Authentication is required to grant an application high priority scheduling/,+3 s/no/yes/' test
```

To make the change inline, use the `-i` flag in addition to `-E` for extended regexp.

### Encrypt YubiKey With Static Password

This is useful for any type of repetitive task where standard keyboard injection is not viable.
Think of a [bad usb](https://en.wikipedia.org/wiki/BadUSB) but quickly re-programmable
for system admin purposes.

First install the `ykman` application.

```none
sudo apt install yubikey-manager
```

Then you man perform the following to determine the serial number.

```none
$ ykman list
YubiKey 5 NFC (5.4.3) [OTP+FIDO+CCID] Serial: 1234567
```

Using the discovered serial number, imprint the password `sysadmin`.

```none
ykman -d 123456 otp static --keyboard-layout US 1 "sysadmin"
```

### Make a JPG Bigger

But WHY!!! This is useful for websites that require a minimum file size for example.

You can use `dd` to cat some zeroes to the end of the file and it still counts as a JPG

```bash
# make the image 100KB bigger
dd if=/dev/zero bs=1024 count=100 | cat >> IMAGE.jpg
```

### SSH and SFTP jump host

You can use an intermediary host as a jump host.

```none
+---+      +----+      +---+
|PC1|----->|JUMP|----->|PC2|
+---+      +----+      +---+
\1.2.3.4    \10.0.0.1   \192.168.0.100
             \port 22    \port 23
```

```none
jumphost -J bastion@10.0.0.1:22 roland@192.168.0.100:23
```

Using the `-J` flag on modern versions of `sftp` (>=8.0) can also use a jump host in the same way
as above. If the version of openssh is not >=8.0 you can use the following command.

```none
sftp -o ProxyJump=bastion@10.0.0.1:22 roland@192.168.0.100:23
```

### Show IP Information For One Interface

For example the device name could be: ens192, wlp3s0, lo

```none
ip add show dev <device name>
```

### Remap CAPS to ESC in X

Below rebinds CAPS as ESC.

This is useful for vim where escape brings you back to normal mode.

Only works in Xorg unfortunately.

```none
setxkbmap -option caps:escape
```

### Remap CAPS to ESC in Wayland

Similar to above, this sway config change will work for wayland/sway.

```none
xkb_options caps:escape
```

### Get TB Written to a Drive

Assuming 512 byte sectors. Change `sdX` as appropriate.

```none
sudo smartctl -A /dev/sdX | awk '/^241/ { print "TBW: "($10 * 512) * 1.0e-12, "TB" } '
```

### Find Linux English Name For Key

Use the `xev` command with `grep`.

```none
❯ xev | grep keysym
    state 0x0, keycode 36 (keysym 0xff0d, Return), same_screen YES,
    state 0x0, keycode 50 (keysym 0xffe1, Shift_L), same_screen YES,
                                           \ name of key                                           
```

### Paste to vim command buffer

When typing a vim command, you can paste your clipboard with `<C-r> "` (ctrl + r  + ")

This also works for registers that are not `"`.
For example you can copy text to `e` and then press `<C-r> "`.

### Protect a File/Folder From Accidental Deletion

To add protection write `sudo chattr +a -R <folder/file>`.

* The file can now only be appended to. Ie with `>>` redirection.
* The `-R` flag makes it recursive.

* To remove protection write `sudo chattr -a <folder/file>`.
* The file has the -a (append only) flag removed.

### Git Force Push

Sometimes you get yourself in a situation where you just want to start over again and go back to a git commit 'checkpoint', force push can do this with the following commands.

First determine the commit hash you want to go back to with the `git log` command.

The git log has most recent at the top, gg will also bring you to the top and most recent, G will bring you to the oldest commit.

Once you have the commit hash you can use this.

```none
git reset --hard <commit hash>
git push -f origin HEAD
```

### Git Push to Branch Without Specifying Remote

When you create a new branch, on your first push github will ask you to specify a remote.

```none
❯ git push
fatal: The current branch add-sass-dep has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin add-sass-dep
```

This can be avoided by using this git incantation while on the branch you want to push.

```none
git push -u origin HEAD
```

### Mongo Shell Count Distinct

I want to count the unique `iso_codes` in my mongodb collection.

```none
docker exec -it container_name bash

mongo
> use database_name
> db.<collection_name>.distinct('column_name')
```

### Dockerfile Multi Stage Builds

A useful docker feature is multi state builds where you can copy artifacts from one container to another within the same dockerfile.

Example using `--copy` to copy an artifact.

```dockerfile
FROM node:16 as build
...
RUN ["npm", "run", "build"]

FROM nginx:latest as web
COPY --from=build /usr/src/app/dist /usr/share/nginx/html
```

Example of using named stages to conditionally build the container (think of it as an IF statement)

```dockerfile
FROM nginx:latest as core
...

FROM core as development
...

FROM core as production
...
```

With docker-compose you can target the different stages.

```yaml
services:
  nginx_proxy:
  build:
    context: .
      dockerfile: dockerfile
      target: production
```

### Vim Macro

Press `q` then a letter between `a` and `z`, this will be your register.

Then type your macro, and press `q` when you are done.

To run your macro press `@` and then the register letter you used.

### Set Key Repeat Rate

Useful to make the caret move faster when navigating text.

```none
xset r rate 280 40
```

### Dump Postgresql Database

On modern versions (>8.4) of Postgresql, you can dump a database to a file.

```none
pg_dump --column-inserts --data-only --table=<table> <database>
```

### Make an IP Static on a Linux Server

Edit `/etc/systemd/network/20-wired.network`.

```none
[Match]
Name=ens192

[Network]
Address=10.0.0.11/24
Gateway=10.0.0.1
DNS=10.0.0.10
```

```none
sudo systemctl enable --now systemd-networkd.service
```

### Extract a Subset of Pages From a PDF

```none
pdftk input.pdf cat 12-15 output output.pdf
```

### Typescript Create Extend Interface and Remove Fields

Use the `Omit` keyword.

```ts
interface IPerson {
    name: string;
    likes: string;
    age: number;
}
// { name: string; likes: string; age: number }

interface ISimplePerson extends Omit<IPerson, "likes">, Omit<IPerson, "age"> {};
// { name: string; }
```

### Fix SSH Asking for a Password

Even though you ran `ssh-add -i ~/.ssh/id_mykey user@server` ssh still asks for a password.

Maybe you don't have the SSH agent running? This is particular the case when running as root.

```none
ssh-add ~/.ssh/id_borg
```

```output
Could not open a connection to your authentication agent.
```

Fix by starting the ssh agent.

```none
eval $(ssh-agent)
ssh-add ~/.ssh/id_borg
```

```output
Agent pid 1581
Identity added: /root/.ssh/id_borg (root@nginx)
```

### Running a Script as sudo Does not Preserve Exports

This behaviour...

```bash
export MYVAR="world"
sudo "echo hello $MYVAR"
```

```output
sudo: echo hello world: command not found
```

Is caused by two things.

1. Environment is weird (see sudos preserve_env)
2. The use of the quotes.

One fix is to write it like this instead.

```bash
export MYVAR="world"
sudo echo "hello $MYVAR"
```

But in some situations, another thing to try is to use the `-E` flag to preserve the enviroment.

```bash
export MYVAR="world"
sudo -E bash -c "echo hello $MYVAR"

# -E = preserve env
# bash = then run bash
# -c = the -c option executes the commands from a string
```

```output
hello world
```
