# Docker Remote Endpoints for Portainer

## Step 1 - Installing docker

```none
sudo apt-get remove docker docker-engine docker.io containerd runc
```

```none
sudo apt-get update
```

```none
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```

```none
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
```

```none
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"
```

Fall back to `stretch` if you get an error from apt when running `sudo apt update`.

```none
# /etc/apt/sources.list
deb [arch=amd64] https://download.docker.com/linux/debian stretch stable
```

```none
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

Then check the status.

```none
systemctl start docker
systemctl enable docker
systemctl status docker
```

## Step 2 - Starting portainer

First get the portainer CE image.

```none
docker pull portainer/portainer-ce
```

Then we need to start the container, I prefer starting with docker-compose because its easier to document and read.

```yaml
version: '3'
services:
    portainer:
        image: portainer/portainer-ce
        container_name: portainer
        volumes:
        - /var/run/docker.sock:/var/run/docker.sock
        - /opt/portainer:/data
        ports:
        - 9000:9000
        restart: always
```

A good optional modification to make is to expose portainer locally to your host, and reverse proxy to it through a gateway (like nginx)

```yaml
ports:
    - 127.0.0.1:9000:9000
```

Then start portainer!

```none
docker-compose up -d
```

## Step 3 - Exposing the docker API

**DANGER** This exposes your docker to the world unsafely **DANGER**

```none
sudo vim /etc/systemd/system/multi-user.target.wants/docker.service
```

Change this...

```none
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```

To this...

```none
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:
```

Then reload and restart.

```none
sudo systemctl daemon-reload
sudo systemctl restart docker
```

Test the connection from the remote client that wants to access your newly exposed docker API with this command.

```none
docker -H 1.2.3.4:2375 info
```

Lastly, make sure to expose this port on your firewall.

## Step 4 - Securing the API

Make sure to read more about portainer endpoints and securing the docker API here.

https://docs.docker.com/engine/security/protect-access/

https://medium.com/trabe/using-docker-engine-api-securely-584e0882158e

https://lemariva.com/blog/2019/12/portainer-managing-docker-engine-remotely

There are 2 official ways of securing dockers API, SSH, and TLS, i decided to use TLS because SSH didn't work for me for some reason.

Another idea i had was to put the communication in a wireguard tunnel somehow. But i haven't read much about this yet.
If i wanted to look into this i would need to understand this https://youtu.be/88GyLoZbDNw?t=1142 (@19:05 onwards)
about networking namespaces most likely.

## Securing with TLS

Based on trabes article [here](https://medium.com/trabe/using-docker-engine-api-securely-584e0882158e), TLS is a good approach to securing dockers API.

In this situation we have two machines:

* My local LAN network hosting a docker daemon on 10.0.0.50
* My remote docker network on my VPS (which also hosts portainer) on 1.2.3.4

We need to do the following to succeed at securing docker.

* Expose port 2376 for secure traffic (2375 is traditionally for unsecure traffic exposed by dockers api)
* Generate a TLS server certificate authority to sign a...
  * Server certificate
  * Client certificate
* Associate the client certificate with portainer to authenticate against my local LAN networks docker daemon

create a location to put these certs.

```none
mkdir ~/portainer-certs
cd portainer-certs
```

Generate a CA certificate on the host machine you want to expose the api from (in my case 10.0.0.50).

You must fill the Common Name field with the FQDN of the docker host machine.

Here i am picking store.rolandw.lan for my FQDN because while my portainer instance is running elsewhere on portainer.rolandw.dev (.dev instead of .lan). **The CA FQDN should be picked for for the host its generated on** even if its not reachable via the internet. So you will most likely be picking some.example.lan or some.example.local.

```none
openssl genrsa -aes256 -out ca-key.pem 4096
```

```none
openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem
Country Name (2 letter code) [AU]:AU
Locality Name (eg, city) []:Melbourne
Common Name (e.g. server FQDN or YOUR name) []:store.rolandw.lan
```

Create a certificate signing request (CSR). Make sure to **replace the CN with your own**.

```none
openssl genrsa -out server-key.pem 4096
openssl req -subj "/CN=store.rolandw.lan" -sha256 -new -key server-key.pem -out server.csr
```

Then sign the CSR with the CA.

```none
openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out server-cert.pem
```

Before generating the server certificate, we need to create an options file that defined the IP and domain names for our server.

```none
echo subjectAltName = \
DNS:store.rolandw.lan,IP:10.0.0.50,IP:127.0.0.1 >> extfile.cnf
```

```none
echo extendedKeyUsage = serverAuth >> extfile.cnf
```

Then generate the server cert.

```none
openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem \
-CAkey ca-key.pem -CAcreateserial -out server-cert.pem \
-extfile extfile.cnf
```

Now we need to generate a client certificate.

```none
openssl genrsa -out key.pem 4096
```

```none
openssl req -subj '/CN=client' -new -key key.pem -out client.csr
```

```none
echo extendedKeyUsage = clientAuth > extfile-client.cnf
```

```none
openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem \
-CAkey ca-key.pem -CAcreateserial -out cert.pem \
-extfile extfile-client.cnf
```

Lastly, docker needs to be started with TLS. Change the exec start in your docker service file to this, you back up the original ExecStart by copying and commenting it out. Also change your paths for each file specified to the location you created your own certificates.

```none
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2376 --tlsverify --tlscacert=/home/roland/portainer-certs/ca.pem --tlscert=/home/roland/portainer-certs/server-cert.pem --tlskey=/home/roland/portainer-certs/server-key.pem
```

Then all you need to do is from portainer, navigate to `endpoints -> add endpoint` and add a `docker endpoint` (connect directly to docker api).

* Name: whatever you want
* Endpoint URL: 1.2.3.4:2376
* Public IP: 1.2.3.4
* TLS: yes
  * Select the 2nd TLS mode (TLS with client verification only)
  * TLS certificate: cert.pem
  * TLS key: key.pem
