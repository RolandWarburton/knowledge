# Systemd Credentials

A common problem in automation is how to keep a secret.
Holding secrets on a linux machine can be risky if done incorrectly.

How can you store a secret that is:

* Not in clear text
* Can only be read by privileged processes
* Does not need a password to retrieve it

Systemd provides a way to achieve this by using systemd-creds.

The idea is to use TPM to de-encrypt a secret stored on disk and use
the systemd privileged access to read it.

Ideally this file is stored on encrypted storage so at rest
its hardened against physical access to the machine.

## Install Requirements

Ensure you have TPM available (TPM2 specifically).

```bash
# check what support you have
systemd-analyze has-tpm2
```

```bash
# install any missing packages
sudo apt install \
    tpm2-tools \
    tpm2-abrmd \
    libtss2-esys-3.0.2-0t64 \
    libtss2-mu-4.0.1-0t64 \
    libtss2-rc0t64
```

```bash
# start the Access Broker and Resource Manager Daemon
systemctl enable tpm2-abrmd
```

## Creating and Decrypting Secrets

You will need a privileged account to generate a secret.

```bash
echo -n "my_secret" | \
    sudo systemd-creds encrypt \
    --tpm2-device=auto - /etc/credstore/my_secret
```

Retrieving the secret **outside** of a systemd unit
will also require privileged access.

```bash
sudo systemd-creds decrypt /etc/credstore/my_secret
# > my_secret
```

## Accessing Secrets in Unit Files

For the purpose of demonstration lets create a script we want to execute.


```bash
#!/usr/bin/bash

# SCRIPT PATH:
# /usr/local/bin/backup.sh

cat "${CREDENTIALS_DIRECTORY}/my_secret"

# rest of script...
# ...
# ...
```

And the corresponding unit file.

```ini
[Unit]
Description=Example Service

[Service]
ExecStart=/usr/local/bin/backup.sh
User=my_user
LoadCredentialEncrypted=my_secret:/etc/credstore/my_secret

[Install]
WantedBy=multi-user.target
```

You can now observe the de-encrypted password using journalctl.

```bash
sudo journalctl -u restic-backup.service --since "10m ago"
```
