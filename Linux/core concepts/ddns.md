# Dynamic DNS (DDNS)

It may be useful to have your computer identifiable via a FQDN at all times,
even if the IP of your device changes.

To create this effect you will need:

* A DNS service (cloudflare) to resolve your device laptop.example.com
  * Zone ID: CF dash -> select example.net -> scroll down the right sidebar on the Overview page
  * API Key: dash.cloudflare.com/profile/api-tokens -> Create Token -> use the "Edit zone DNS" template -> scope it to example.net.
* Make sure you create an A record in your DNS service first
* A script that calls the cloudflare API to update the zone information
* A way to schedule the script to run over and over


## Create the A record

Make sure:

* The A record must exist in Cloudflare before the script will work
* TTL is set to 60 seconds so IP changes propagate quickly

## Create the DDNS script

Create a your DDNS script.

<details>
<summary>DDNS script `/usr/local/bin/ddns.sh`</summary>

```bash
#!/bin/bash
set -euo pipefail

RECORD_NAME="laptop.example.net"
ZONE_ID="${ZONE_ID}"
API_TOKEN="${API_TOKEN}"

# Get current public IP
IP=$(curl -sf https://ifconfig.co) || { echo "Failed to get IP"; exit 1; }

# Get the DNS record ID and current IP
RECORD=$(curl -sf "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/dns_records?name=${RECORD_NAME}&type=A" \
  -H "Authorization: Bearer ${API_TOKEN}")

RECORD_ID=$(echo "$RECORD" | jq -r '.result[0].id')
CURRENT_IP=$(echo "$RECORD" | jq -r '.result[0].content')

if [[ "$IP" == "$CURRENT_IP" ]]; then
  echo "IP unchanged ($IP), skipping update"
  exit 0
fi

# Update the record
curl -sf -X PUT "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/dns_records/${RECORD_ID}" \
  -H "Authorization: Bearer ${API_TOKEN}" \
  -H "Content-Type: application/json" \
  --data "{\"type\":\"A\",\"name\":\"${RECORD_NAME}\",\"content\":\"${IP}\",\"ttl\":60,\"proxied\":false}"

echo "Updated ${RECORD_NAME} to ${IP}"
```

</details>

## Create the secrets file

Next create a location to store the Cloudflare secrets `/etc/credentials/ddns.env`.
Make sure the environment values are not quoted.

```none
ZONE_ID=PLACEHOLDER
API_TOKEN=PLACEHOLDER
```

Make sure to secure the file:

```bash
sudo chmod 600 /etc/credentials/ddns.env
```

## Create the service and timer files

Next create the service and timer files

```bash
# /etc/systemd/system/ddns.service
[Unit]
Description=Update Cloudflare DNS for laptop.example.net
After=network-online.target
Wants=network-online.target

[Service]
Type=oneshot
EnvironmentFile=/etc/credentials/ddns.env
ExecStart=/usr/local/bin/ddns.sh
```

```bash
# /etc/systemd/system/ddns.timer
[Unit]
Description=Run Cloudflare DDNS update every 5 minutes

[Timer]
OnBootSec=30
OnUnitActiveSec=5min

[Install]
WantedBy=timers.target
```
