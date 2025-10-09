# WireGuard

## Quick Setup

```bash
# install WireGuard
sudo apt install wireguard resolvconf
```

WireGuard config file names should be named after the interface they represent.

Most private P2P connections should use `wg0` as the interface name.
VPN connections should use the VPN server name (for example `sg-sin-wg-001`).

## Generate Keys

Each host has a public and private key.

```bash
wg genkey > private
wg pubkey < private > public
```

## Configuration Example

For this example our two peers are

* 92.175.34.88
* 137.42.196.157

```bash
# /etc/wireguard/<interface_name>.conf
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = iNUNhy/S41mvw/W/VJ4qpG7lmZDCjwvLMXlfbToGJ2g=

[Peer]
PublicKey = IXrhGRzw3YeTSkS1arbbmdd4qNbJ14/RhOenXcGSozg=
Endpoint = 92.175.34.88:51820
AllowedIPs = 10.0.0.2/32
```

```bash
# /etc/wireguard/<interface_name>.conf
[Interface]
Address = 10.0.0.2/24
ListenPort = 51820
PrivateKey = eIa5vYTXhQUZFQv2pvZWNWTqwyo2fakiXwAz7U3JwF4=

[Peer]
PublicKey = PmpUW9+apRLKS1GiSplvCLyjanCl7H244gVhvSgdslE=
Endpoint = 137.42.196.157:51820
AllowedIPs = 10.0.0.1/32
```

## Mullvad

Log in to Mullvad and generate a config [here](https://mullvad.net/en/account/wireguard-config).
