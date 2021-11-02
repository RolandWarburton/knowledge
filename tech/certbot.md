# Cerbot

Get nginx.

```none
sudo apt install nginx
```

Then configure a domain, i am provisioning [*.rolandw.dev](https://borg.rolandw.dev)

```none
sudo apt install certbot python3-certbot-nginx python3-certbot-dns-cloudflare

sudo certbot --nginx
```

Go get the credentials file from [https://cloudflare.com](https://cloudflare.com)

![cf_01](https://i.imgur.com/b2IKNM3.png)

![cf_02](https://i.imgur.com/EOfPZyp.png)

![cf_03](https://i.imgur.com/2Ia1VhI.png)

![cf_04](https://i.imgur.com/GeYkRdz.png)

![cf_05](https://i.imgur.com/SNJNlDT.png)

Test your token

```none
curl -X GET "https://api.cloudflare.com/client/v4/user/tokens/verify" \
     -H "Authorization: Bearer TOKEN" \
     -H "Content-Type:application/json"
```

Create a config file called `credentials.ini`

```none
# ~/credentials.ini

# Cloudflare API token used by Certbot
dns_cloudflare_api_token = YOURTOKENHERE
```

```none
chmod 600 ~/credentials.ini
```

The path to this file can be provided using the `--dns-cloudflare-credentials ./credentials.ini` argument in the certbot script below.

```none
# ~/certbot.sh

sudo certbot certonly \
  --dns-cloudflare \
  --dns-cloudflare-credentials ./credentials.ini \
  -d *.rolandw.dev \
  -d rolandw.dev
```

```output
nginx@nginx:~$ ./certbot.sh
bash: warning: setlocale: LC_ALL: cannot change locale (en_US.UTF-8)
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator dns-cloudflare, Installer None
Requesting a certificate for *.rolandw.dev and rolandw.dev
Performing the following challenges:
dns-01 challenge for rolandw.dev
dns-01 challenge for rolandw.dev
Waiting 10 seconds for DNS changes to propagate
Waiting for verification...
Cleaning up challenges

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/rolandw.dev/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/rolandw.dev/privkey.pem
   Your certificate will expire on 2022-01-08. To obtain a new or
   tweaked version of this certificate in the future, simply run
   certbot again. To non-interactively renew *all* of your
   certificates, run "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

To renew certs. Certbot will pick up on the certificates and knows to look for the credentials.ini file in the same location that you put it originally.

```none
sudo certbot renew --dry-run
```

Certbot stores information about this domain in `cat /etc/letsencrypt/renewal/rolandw.dev.conf`.

I changed the `dns_cloudflare_credentials` to be the full path just in case for next time, and running the renewal in scripts.

```none
# renew_before_expiry = 30 days
version = VERSION_NUMBER
archive_dir = /etc/letsencrypt/archive/rolandw.dev
cert = /etc/letsencrypt/live/rolandw.dev/cert.pem
privkey = /etc/letsencrypt/live/rolandw.dev/privkey.pem
chain = /etc/letsencrypt/live/rolandw.dev/chain.pem
fullchain = /etc/letsencrypt/live/rolandw.dev/fullchain.pem

# Options used in the renewal process
[renewalparams]
account = ACCOUNT_NUMBER
authenticator = dns-cloudflare
dns_cloudflare_credentials = /home/nginx/credentials.ini
server = https://acme-v02.api.letsencrypt.org/directory
```

## Certbot in docker

Sometimes weird things happen when installing software, for that we have docker!

Heres an example from a script where i deploy a certificate for my blog website, we need to bind mounts to the container so that we can access the certificates created, another good way of doing things is to mount a volume and place the certs in there, that way another nginx container can access easily in the future and its quite easy to run another docker run script to renew certs as well.

```none
sudo docker run -it --rm --name certbot \
    -v "/etc/letsencrypt:/etc/letsencrypt" \
    -v "/var/lib/letsencrypt:/var/lib/letsencrypt" \
    -v "/home/roland/cloudflare:/creds" \
    certbot/dns-cloudflare certonly \
    --agree-tos \
    --email warburtonroland@gmail.com \
    -d blog.rolandw.dev \
    --dns-cloudflare \
    --dns-cloudflare-credentials /creds/credentials.ini
```
