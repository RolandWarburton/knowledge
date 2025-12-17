# Postfix via Amazon SES

Lab notes configuring a VPS for sending mail.

* Server: VPS (vultr)
* Domain: wirecrop.net
* Mail From Domain: mail.wirecrop.net
* AWS region: ap-southeast-2

## Prepare the host

Mail providers use rDNS (reverse DNS) to check the name of the server.
If the name does not match, emails are likely to be rejected.

We need to set the rDNS to be the **Mail from Domain**.

* Navigate to vultr (or your VPS)
* Select your server
* Select the server settings
* Look for **ipv4** configuration
* find **reverse DNS**
* set **reverse DNS** to mail.wirecrop.net

Next we need to install dependencies

```bash
# postfix MTA (mail transfer agent)
sudo apt install postfix
# for postfix authentication with aws ses
sudo apt install libsasl2-modules
```

## Configuring SES

Navigate to the AWS console.
Search for and open "Amazon Simple Email Service".

Under `Configuration -> Identities` Create a new identity.

* type: Domain
* domain: wirecrop.net
* DKIM: Easy DKIM

Configure the identity.

* Set the **Custom MAIL FROM domain** to mail.wirecrop.net
* Publish the records under **Publish DNS records** to cloudflare (or alternative DNS provider)
* wait for the identity to become verified (30min)

Generate SMTP credentials.

* Find **SMTP settings** in the SES navigation tree
* Create SMTP credentials from here
* [reference](https://docs.aws.amazon.com/ses/latest/dg/smtp-credentials.html)

## Configuring postfix

[reference](https://docs.aws.amazon.com/ses/latest/dg/postfix.html).

Configure postfix settings.

```bash
# delete the contents of /etc/postfix/main.cf first
sudo postconf -e "relayhost = [email-smtp.ap-southeast-2.amazonaws.com]:587" \
"smtp_sasl_auth_enable = yes" \
"smtp_sasl_security_options = noanonymous" \
"smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd" \
"smtp_use_tls = yes" \
"smtp_tls_security_level = secure" \
"smtp_tls_note_starttls_offer = yes"

# Security settings
sudo postconf -e \
"smtpd_relay_restrictions = permit_mynetworks, permit_sasl_authenticated, reject_unauth_destination" \
"smtpd_recipient_restrictions = permit_mynetworks, permit_sasl_authenticated, reject_unauth_destination"

# rewrite rule
sudo touch /etc/postfix/generic # we will add content later
sudo postconf -e \
"smtp_generic_maps = hash:/etc/postfix/generic"

# tell Postfix where to find the CA certificate.
sudo postconf -e 'smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt'
```

Set SASL password. Generated when you generated SMTP credentials.

```bash
# /etc/postfix/sasl_passwd
[email-smtp.ap-southeast-2.amazonaws.com]:587 USERNAME:PASSWORD
# Example: AKIASDKLJHASLKDJH:n7V+p9X2ZkLqW1m5N8yB4xR0vC3zA6jE
```

Mask the local address with the real one when sending emails from the VPS.

```bash
# /etc/postfix/generic
roland@pluto.guest mail@mail.wirecrop.net
```

Create the database files for sasl_passwd, and generic.

```bash
sudo postmap -v hash:/etc/postfix/sasl_passwd
sudo postmap -v hash:/etc/postfix/generic

sudo chown root:root \
    /etc/postfix/sasl_passwd \
    /etc/postfix/generic \
    /etc/postfix/generic.db
```

Restart postfix service.

```bash
sudo systemctl restart postfix
```

The quickest way to test mail is to use the `sendmail` command.

```bash
sudo /usr/sbin/sendmail -f mail@wirecrop.net example@gmail.com <<EOF
From: Wirecrop Notifications <mail@wirecrop.net>
To: example@gmail.com
Subject: Test Email from Postfix/Sendmail CLI

This is a test message sent manually from the command line using the sendmail utility.
.
EOF
```

## Diagnosing

```bash
# check the postfix queue
sudo postqueue -p

# delete the postfix queue
sudo postsuper -d ALL deferred

# check its health
sudo journalctl --since="1min ago" -u postfix

# check other log files
sudo tail /var/log/syslog
sudo tail /var/log/mail.log
```
