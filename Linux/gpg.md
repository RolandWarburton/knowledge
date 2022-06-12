# GPG Basics

## Creating a key

`gpg --full-generate-key`

## Viewing GPG key

```none
gpg --list-keys
```

* You can see your key "pub" ID which is 41 chars long.
* You can see the name of the owner of the key

## Exporting a key

The `-a` flag represents ascii text.

```none
gpg --export -a 1CB1AADD3F35902234887B922C1C6E0C75E4CC05 > public.key
```

```none
gpg --export-secret-key -a 1CB1AADD3F35902234887B922C1C6E0C75E4CC05 > private.key
```

## Importing a key

```none
gpg --import public.key
```

```none
gpg --import private.key
```

Once you have imported a public key, you need to set its trust.

By default your own keys are `[ultimate]` trust level.

```none
gpg --edit-key 1CB1AADD3F35902234887B922C1C6E0C75E4CC05
> trust
> 5
> quit
```

## Encrypting a file

```none
 gpg --encrypt --output document.gpg --recipient 1CB1AADD3F35902234887B922C1C6E0C75E4CC05 document
```

## Decrypting a file

To stdout

```none
gpg --decrypt document.gpg
```

Write to a file

```none
gpg --decrypt --output document document.gpg
```

