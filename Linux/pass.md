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

### Add additonal information

passwords often need additional information, such as secret questions, or pin numbers. You can use `pass edit <entry>` to add this information. You can add any information you want and it will be printed when you retrieve the password using `pass <entry>`. For example.

```none
❯ pass test.com
password123
Secret Question 1: what did you eat for breakfast? none ya business.
Support pin: 1234
email: joe@mama.com
```

## Intergrating with git

```none
git init --bare ~/.password-store
pass git init
```

Then add the remote origin using pass.

```none
pass git remote add origin git@github.com:RolandWarburton/password-store.git
```

Then push your changes.

```none
pass git push --set-upstream origin master
```

