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

Nothing here is guranateed to work, or even be useful in the long term.
Just a collection of computer "spells" that i have used more than once.

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
