# Pass - UNIX Password Manager

## Generate a key

```none
gpg --full-gen-key
```

```output
gpg (GnuPG) 2.2.20; Copyright (C) 2020 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
  (14) Existing key from card
Your selection? 1
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (3072) 4096
Requested keysize is 4096 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 0
Key does not expire at all
Is this correct? (y/N) y

GnuPG needs to construct a user ID to identify your key.

Real name: Roland
Email address: warburtonroland@gmail.com
Comment:
You selected this USER-ID:
    "Roland <warburtonroland@gmail.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? N
Real name: Roland Warburton
You selected this USER-ID:
    "Roland Warburton <warburtonroland@gmail.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
gpg: key B1132B17EC2146A1 marked as ultimately trusted
gpg: revocation certificate stored as '/home/roland/.gnupg/openpgp-revocs.d/1234567812345678123456781234567812345678.rev'
public and secret key created and signed.

pub   rsa4096 2021-01-27 [SC]
      1234567812345678123456781234567812345678
uid                      Roland Warburton <warburtonroland@gmail.com>
sub   rsa4096 2021-01-27 [E]
```

```none
gpg --list-keys
```

```output
pub   rsa4096 2021-01-27 [SC]
      1234567812345678123456781234567812345678
uid           [ultimate] Roland Warburton <warburtonroland@gmail.com>
sub   rsa4096 2021-01-27 [E]
```

In this example the key is `1234567812345678123456781234567812345678`

If you messed up you can delete the key using `gpg --delete-secret-keys <tab>` to remove the selected key.

## Generate pass database

Using the gpg key above.

```none
pass init 1234567812345678123456781234567812345678
```

```output
❯ pass init 1234567812345678123456781234567812345678
mkdir: created directory '/home/roland/.password-store/'
Password store initialized for 1234567812345678123456781234567812345678
```

## Managing pass basics

### Adding a new login

```none
pass insert domain.com
```

Most of the time you will want to insert a password
with additional information like the username and email.

Use `m` for multiline insertion. This will open with your `$EDITOR`.

```none
pass insert -m domain.com
```

You can also specify multiple passwords in a file structure.

```none
pass insert domain.com/login1
pass insert domain.com/login2
```

### Viewing logins

Just type `pass` to see your logins in a tree structure

```output
❯ pass
Password Store
└── test.com
```

### Delete a login

Use `pass delete <entry>`.

```none
❯ pass delete test.com
Are you sure you would like to delete test.com? [y/N] y
removed '/home/roland/.password-store/test.com.gpg'
```

### Add additional information

passwords often need additional information, such as secret questions, or pin numbers. You can use `pass edit <entry>` to add this information. You can add any information you want and it will be printed when you retrieve the password using `pass <entry>`. For example.

```none
❯ pass test.com
password123
Secret Question 1: what did you eat for breakfast? none ya business.
Support pin: 1234
email: joe@mama.com
```

## Integrating with git

```none
pass git init
```

Then add the remote origin using pass.

```none
pass git remote add origin git@github.com:RolandWarburton/password-store.git
```

Then push your changes.

```none
pass git push -u --all
```

Or pull your changes (if prompted to merge just `:q` when the merge editor comes up)

```none
pass git pull origin master --allow-unrelated-histories
```

```output
From github.com:RolandWarburton/password-store
 * branch            master     -> FETCH_HEAD
Merge made by the 'recursive' strategy.
 test.com.gpg | Bin 0 -> 587 bytes
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 test.com.gpg
```

## Syncing password database

### Export PGP keys

Export your public and private key. These keys pgp keys will be placed in your home directory.

The reason why these keys end in `.pgp` and not `.gpg` is because gpg is gnu software that implements the pgp technology, so the raw key during transit to our new computer will be a pgp key.

The reason why `--armor` is used is to change the file format of the resulting file to be in "armored ascii" rather than a binary file, which makes transport of the exported key file easier.

To export your public key.

```none
gpg --output public.pgp --armor --export warburtonroland@gmail.com 
```

To export your private key (CAREFUL NOT TO SHARE THIS ONE).

```none
gpg --output private.pgp --armor --export-secret-key warburtonroland@gmail.com
```

### Import PGP keys

Next place the key on your remote computer, for example through sftp.

```output
sftp roland@remote_computer
> put .private.pgp
```

Then log into that machine over ssh and import the key.

```none
gpg --import private.pgp
```

You will be prompted to enter your master password and the result output of `gpg --list-keys` should print your key, your key will also likely be considered `[unknown]` instead of `[ultimate]` which indicates that this key is not trusted yet (see the trust database at `~/.gnupg/trustdb.gpg`) ([source](https://unix.stackexchange.com/questions/407062/gpg-list-keys-command-outputs-uid-unknown-after-importing-private-key-onto)). To fix this issue, use the following commands.

```none
gpg --edit-key user@useremail.com
```

```output
❯ gpg --edit-key warburtonroland@gmail.com
gpg (GnuPG) 2.2.12; Copyright (C) 2018 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Secret key is available.

sec  rsa4096/B1132B17EC2146A1
     created: 2021-01-27  expires: never       usage: SC
     trust: unknown       validity: unknown
ssb  rsa4096/3AFDF5CA8F188A62
     created: 2021-01-27  expires: never       usage: E
[ unknown] (1). Roland Warburton <warburtonroland@gmail.com>

gpg> trust
sec  rsa4096/B1132B17EC2146A1
     created: 2021-01-27  expires: never       usage: SC
     trust: unknown       validity: unknown
ssb  rsa4096/3AFDF5CA8F188A62
     created: 2021-01-27  expires: never       usage: E
[ unknown] (1). Roland Warburton <warburtonroland@gmail.com>

Please decide how far you trust this user to correctly verify other users' keys
(by looking at passports, checking fingerprints from different sources, etc.)

  1 = I don't know or won't say
  2 = I do NOT trust
  3 = I trust marginally
  4 = I trust fully
  5 = I trust ultimately
  m = back to the main menu

Your decision? 5
Do you really want to set this key to ultimate trust? (y/N) y

sec  rsa4096/B1132B17EC2146A1
     created: 2021-01-27  expires: never       usage: SC
     trust: ultimate      validity: unknown
ssb  rsa4096/3AFDF5CA8F188A62
     created: 2021-01-27  expires: never       usage: E
[ unknown] (1). Roland Warburton <warburtonroland@gmail.com>
Please note that the shown key validity is not necessarily correct
unless you restart the program.
```

### Init git on the remote system

```none
pass init 1234567812345678123456781234567812345678
```

```none
pass git init
```

```none
pass git remote add origin git@github.com:RolandWarburton/password-store.git
```

```none
pass git pull origin master --allow-unrelated-histories
```

## Diagnosing Issues

### Freezing when retrieving passwords

If you try and run `pass example.com` and the program freezes, there are a couple things to check.

* Is dbus running? `systemctl --user status dbus`.
* Is `DBUS_SESSION_BUS_ADDRESS` set? If not
`export DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/$(id -u)/bus`.
* Is the user in the `dbus` group. `groupadd dbus && usermod -aG dbus roland`.
* Do you have `pinentry pinentry-tty pinentry-curses` installed?.
* Is the default pinentry program set in `~/.gnupg/gpg-agent.conf`?
Use `pinentry-program /usr/bin/pinentry-curses` then restart `gpgconf --kill gpg-agent`.

Test the correct functioning of gpg by trying to encrypt a file.

```none
echo "hello" > test && gpg --symetric test`
```

