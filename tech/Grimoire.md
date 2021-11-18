# Grimoire

From [Fred Bednarski](https://fdisk.space/grimoire/), a grimoire is a collection of useful software bits.

> The grimoire is a scrapbook for useful bits of code,
> software solutions and all kinds of tech wizardry.
> The entries here are something between a reference and a tutorial.

## Forward

Summoning daemons to do our bidding, making lifeless things appear alive,
writing esoteric commands in hermetic languages and tricking rocks into thinking.
Yep, computers are pretty much sorcery.

Nothing here is guranateed to work, or even be useful in the long term.
Just a collection of computer "spells" that i have used more than once.

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
