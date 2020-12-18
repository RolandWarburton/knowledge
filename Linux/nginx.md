# Nginx

## The basics

### Server Blocks

Server blocks are used to define different websites on the same server.

```none
# Server blocks are located in nginx configs
# define a file fot each website
# If you are suing subdomains, they belong in one file along with the website itself

/etc/nginx/sites-available/
├── default
└── roaming.host
```

### Server block files

A server block may be wrapped in `http{...}` but depending on if you should wrap your website in http depends on how your `/var/www/*` is set up.

### Servers

**Only** wrap your servers in a http block if they are inside `/var/www/http/*`

I prefer to structure my websites from the www folder so for this example its not needed to wrap the server in http because its not inside the http directory.

```none
/var/www
├── api.roaming.host # 'root' location for a server block
│   └── index.html
├── html
│   └── index.nginx-debian.html # default nginx server block
└── roaming.host
    └── index.html
```

```none
# Generic server block file
server {
	listen 80 default_server;
	server_name roaming.host default_server;
	root /var/www/roaming.host
	index index.html

	location / {
		try_files $uri $uri.html $uri/ /404;
	}
}
```

### Subdomains

Subdomains belong in separate `server{...}` blocks. Each subdomain for a website should belong inside one configurable file that belongs inside `/etc/nginx/available-sites` and then sym-linked to `enabled-sites`.

```none
# Default Primary server
server {
	listen 80 default_server;
	#		  ^
	#		  |
	#		  default server
	#
	server_name roaming.host default_server;
	root /var/www/roaming.host
	index index.html

	location / {
		try_files $uri $uri.html $uri/ /404;
	}
}

# An api subdomain
server {
	listen 80;
	server_name api.roaming.host;
	#		 	^
	#		 	|
	#		 	note api
	#
	root /var/www/api.roaming.host
	#		 	  ^
	#		 	  |
	#		 	  note different root

	index index.html

	location / {
		try_files $uri $uri.html $uri/ /404;
	}
}
```

### Locations

A location is a way of exposing different end points and doing things to them.

Locations are matched on 4 modifiers

* **no modifier** - prefix match the URI (ie everything past roaming.host/*).
* **=** - Exact match (/one matches /one and not /one/two).
* **~** - If a tilde modifier is present, this location will be interpreted as a case-sensitive regular expression match. ([DI](https://www.digitalocean.com/community/tutorials/understanding-nginx-server-and-location-block-selection-algorithms))
* **~\*** -  If a tilde and asterisk modifier is used, the location block will be interpreted as a case-insensitive regular expression match. ([DI](https://www.digitalocean.com/community/tutorials/understanding-nginx-server-and-location-block-selection-algorithms))
* **^~** - If a carat and tilde modifier is present, and if this block is selected as the best non-regular expression match, regular expression matching will not take place. ([DI](https://www.digitalocean.com/community/tutorials/understanding-nginx-server-and-location-block-selection-algorithms))

#### Location Example - Serve Everything

```none
server {
	listen 80 default_server;
	server_name roaming.host default_server;

	# The base root of the server
	# You can change this value inside locations
	root /var/www/roaming.host

	# Default index
	index index.html

	location / {
		# 1. try to match for a file
		# 2. try to match a html file
		# 3. try to match a directory
		# 4. fail to /404
		try_files $uri $uri.html $uri/ /404;
	}
}
```

Because we have a fallback to 404 (which is required when using try_files to complete the life cycle) its a good idea to place a custom 404 index.html inside `/var/www/roaming.host/404/index.html` that will be served.

You could also change the error code if you choose by using a return value for the `/404` route.

```none
server {
	...
	location / {
		try_files $uri $uri.html $uri/ /404;
	}

	location = /404 {
		# Return file permanently moved instead of not found error
		return 301;
	}
}
```

#### Location Example - Alias Vs Root

Lets say you have a website that looks like this.

```none
/var/www
├── roaming.host # my main website
│   └── index.html
└── secret # super secret page outside of root
    └── index.html
```

You would like to link `/var/www/secret` to your main website when you navigate to `roaming.host/secret`

```none
server {
	...
	# my root location doesn't include my secret file!
	root /var/www/roaming.host

	#I can link my secret file in a special location
	location = /secret {
		alias /var/www/secret/;
	}

	# Serve all my other files
	location / {
		try_files $uri $uri.html $uri/ /404;
	}
}
```

**But why do i need to use alias???**

The difference between alias and root is if the URI is appended or not. Lets say i had the same config except used root instead

```none
# WRONG
location = /secret {
		root /var/www/secret/;
}

# If i use the URL roaming.host/secret
# nginx will interpret it is /var/www/app/secret/secret
# (/secret from the URL is APPENDED)
```

```none
# CORRECT
location = /secret {
		alias /var/www/secret/;
#		^					 ^
#		|					 |
#		use of alias	     trailing slash
}

# nginx will not append /secret from roaming.host/secret
# the evaluated location is now just /var/www/secret and it works
# NOTE the trailing slash is required for an alias
```

### Configuring autoindex (file directory view)

First start by setting up an empty index.html for nginx to populate.

I want the autoindex to be at the `roaming.host/fm` endpoint so i will create the index file inside `/var/www/roaming.host/fm/index.html`. Then put a test file inside that directory for testing.

Then edit your server block to include a new location to enable autoindex for when you browse to that location **only**. You can make it global by putting it outside of `location{...}` but its a security risk.

Make sure to expose the folder by using a slash on both sides (`/fm/`) or nginx will try and match a file instead.

```none
server{
#	make sure you have one default server
	listen 80 default_server;
	server_name roaming.host default_server;

#	try and find an index.html when matching URIs
	index index.html;

	root /var/www/roaming.host;

#	match /fm/ EXACTLY by using '='
#	to ensure that only /fm has autoindex applied to it
# 	use location /fm {...} to see sub folders too
	location = /fm {
		# ignore index.html
		index _;

		# set the directory (leave blank to index the current directory)
		alias

		# enable autoindex
		autoindex on;
	}
}
```

### The basics - Full Config Example

Make sure you have the correct *A records* in your DNS settings on your nameserver. In this example namecheap is the domain host so the @ represents an empty field (ie roaming.host with no subdomain or www).
\

| Type     | Host | Value          | TTL       |
| -------- | ---- | -------------- | --------- |
| A Record | @    | 149.28.174.161 | Automatic |
| A Record | api  | 149.28.174.161 | Automatic |
| A Record | www  | 149.28.174.161 | Automatic |

\

```none
# Roaming Host
server {
	listen 80 default_server;
	server_name roaming.host default_server;
	index index.html;
	root /var/www/roaming.host;

	location = /fm/ {
#		clear the index so it doesn't match anything
		index _;
		alias /var/www/secret/;
		autoindex on;
	}

	location /testing {
		alias /var/www/secret/;
	}

	location / {
		autoindex on;
		try_files $uri $uri.html $uri/ /404;
	}

	location = /500 {
		return 500;
	}

}

# roaming host API
server {
	listen 80;
	index index.html;
	server_name api.roaming.host;
	root /var/www/api.roaming.host;
}
```

### AutoIndex styles

Nginx has the ability to inject files before or after the main body. By using the `add_after_body` or `add_before_body` statement you can point to a html file thats absolute to your location.

In this example my `location = /files` and has an alias to `/home/roland/files` so the style file is placed somewhere inside `/home/roland/files`. The location of the actual file will therefore be `/home/roland/files/.assets/fancyindex.html`.

A gotcha is to make sure to include the the /files in the add_after_body statement.

```none
location /files {
	# go to this dir instead of /files
	alias /home/roland/files;
		# ignore index.html
		index _;
		# enable autoindex
		autoindex on;
		# my own css
		add_after_body /files/.assets/fancyindex.html;
}
```

Heres an example of how your fancyindex.html could look like.

```html
<style>
body {
	background: #121212 !important;
	color: #dedede !important;
	font-size: 1.25em;
}
a, a:visited  {
	color: #dedede;
}
</style>

```

### Theory - The gateway

#### Reverse proxy vs Load balancer

A reverse proxy hides the backend infrastructure from the request. On layer 7 (http) the request is terminated at the rev proxy and a new one is made (proxied) to the target based
on the routing rules present in the reverse proxy (examples below - "Simple upstream backend").

A reverse proxy layer allows for many architectural benefits

* Caching requests
* Canary deployment - you can pick which version of a service a request should go to based on the requests properties, eg for enabling legacy support
* Enhanced security - screening traffic before it reaches your services
* Isolation - isolates services from the internet allowing for more microservice goodness
* Single entry url - lets you play games with urls to run lots of things on one url (when operating on http)
* Load balancing - **all** reverse proxies can be load balancers, load balancers are a subset of reverse proxies when considering features

Furthermore, for better clarification. All gateways are reverse proxies, a reverse proxy hides 1+ services from the internet, and a load balancer is a subset feature of a reverse proxy that provides redundancy for a service by hiding 2+ identical/similar services and distributing load through an algorithm.

A load balancer is a type of reverse proxy (or a feature of a reverse proxy) that allows an algorithm to split traffic between different servers, or otherwise provide some form of
high availability / redundancy for a service or set of services. TLDR a load balancer is a gateway with two or more services sitting behind it, the load balancer knows meta information about
these services and uses an algorithm to distribute traffic.

Because the main difference between if you should be using a load balancer or reverse proxy is mostly based on the structure of the application/s you want to run
Some important differentiating factors to easily identify if an architecture should be a full fat reverse proxy (like nginx) or a load balancer (like HA proxy) is to look at these following properties. Given that these days nginx can provide full layer 7 and layer 4 reverse proxying features, its generally a safer bet, conversely pick HA when you are concerned about data availability, scalability, and serving many requests to a single service at once.

#### Simple upstream backend (round-robin example)

This is part of a gateway architecture pattern for reverse proxying multiple services/applications behind one gateway.

Place this one in `/etc/nginx/nginx.conf`.

By default the upstream algorithm is round-robin.
If you require state in your application (like authentication sessions) then add `ip_hash` to your upstream to make an ip->server connection sticky.

```none
upstream allbackend {
				ip_hash;
                server 127.0.0.1:3001;
                server 127.0.0.1:3002;
        }
```

```none
http {
        upstream allbackend {
                server 127.0.0.1:3001;
                server 127.0.0.1:3002;
        }

        server {
                listen 80 default_server;
                listen [::]:80 default_server;

                root /var/www/html;

                # Add index.php to the list if you are using PHP
                index index.html index.htm index.nginx-debian.html;

                server_name _;

                location / {
                        proxy_pass http://allbackend/;
                }
        }
}
events {

}
```

#### Using multiple backends

Elaborating on the previous codeblock, lets add two upstreams and route to them.

Again, because we are using the http block here, this file is `/etc/nginx/nginx.conf`, not an `sites-avaliable`.

```none
http {
        upstream app1 {
                server 127.0.0.1:3001;
        }

        upstream app2 {
                server 127.0.0.1:3002;
        }

        server {
                listen 80 default_server;
                listen [::]:80 default_server;

                root /var/www/html;

                # Add index.php to the list if you are using PHP
                index index.html index.htm index.nginx-debian.html;

                server_name _;

                location /app1 {
                        proxy_pass http://app1/;
                }

                location /app2 {
                        proxy_pass http://app2/;
                }
        }
}
events {

}
```

## Lets encrypt - HTTPS

This section will be covering what ive learnt about provisioning a website with SSL/TLS certificates through lets encrypt.
I also talk about this under [/notes/writing/Deploying_docker_websites](deploying docker websites) where i take a different approach to applying certificates to sites. In the mentioned notes
i am using a per-site ssl certificate to separate each domain into a more self sustaining site while implementing a nginx reverse proxy through a docker container.

In the following sections i am going over another pattern that i learnt more recently that i believe to be easier to understand and set up for single moderately sized projects, ie. anything hosted on one domain.

### Prerequisites - Ask LE for a certificate

The first step is to install certbot onto your server and acquire a certificate for your domain.
Make sure to use the `--standalone` flag to avoid modification of any configs, we will manually configure it later.

```none
sudo certbot certonly --standalone
```

Then enter your domain name and locate the certificate files.

![generated keys](https://i.imgur.com/ekiUqvI.png)

next place these config options within the `server` block of your config.

```none
ssl_certificate /etc/letsencrypt/live/0x8.host/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/0x8.host/privkey.pem;
```

You can now pretty much run nginx installed with your certificates and any configuration you see above.

Another additional line of configuration you may want to add to your `server` or `https` block is a redirect for all http to https traffic.

```none
server {
	server_name 0x8.host;
	return 301 https://foo.com$request_uri;
}
```

## Docker and Nginx with LE

This config is a little more complex and needs some docker knowledge to set up.

First, lets create some simple backends to implement a reverse proxy architecture. Clone my template containers from [here](https://github.com/RolandWarburton/nodeTidbits/tree/master/proxyStuff/webServers) and spin them up using the provided shell commands. Or if you would like, use your own backend services.

### Building the reverse proxy gateway

Next lets work on the reverse proxy gateway pattern to proxy the requests.

The end goal is to secure our backend `webserver` like this.

```output
+------------+               +-------------+              +-----------+
|    WWW     |  <- https ->  |             |  <- http ->  |           |
| Internet   +---------------+ gateway     +--------------+ webserver |
|            |               |             |              |           |
+------------+               +-------------+              +-----------+
```

Create the following files.

Here is the dead simple dockerfile. Its base image is nginx and we just demonstrate copying over some custom html file.
We also reference a *nginx.conf* that will be created later.

Once you have finished creating all your files, build the container image with `docker build -t gateway .` while inside the directory with the dockerfile.

```docker
FROM nginx:latest

USER root
# place in some custom html stuff here
COPY ./html /usr/share/nginx/html

# place in the custom nginx config
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
EXPOSE 443
```

Here is the *docker-compose.yaml*, here we reference our gateway container we just created, we are exposing the appropriate ports, we are also
using the hosts networking as our `networking mode` which allows us to access our services exposed locally to the host on `127.0.0.1:port`.

Another implementation that i would like to look at is by placing the gateway and service in the same docker network. But this is what works for now so its good enough.

Lastly we mount our certificates into the container, make sure to **change the domain name** of the fullchain/privkey.

```yaml
version: "3.0"
services:
    gateway:
        container_name: "gateway"
        image: gateway
        ports:
            - "80:80"
            - "443:443"
        network_mode: "host"
        volumes:
            - "/etc/letsencrypt/live/0x8.host/fullchain.pem:/etc/letsencrypt/live/0x8.host/fullchain.pem"
            - "/etc/letsencrypt/live/0x8.host/privkey.pem:/etc/letsencrypt/live/0x8.host/privkey.pem"
```

Here is a sample nginx config for reverse proxying our two example docker container [express apps](https://github.com/RolandWarburton/nodeTidbits/tree/master/proxyStuff/webServers).

Pay attention to the domain name in use here as well in nginx.conf. Ive left comments about what each bit is doing. Refer to [this](https://www.youtube.com/watch?v=WC2-hNNBWII) amazing video for more guidance on this config, in particular to understand the basics of reverse proxying and the function of the `upstream` block.

```none
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
        # here we include some default nginx bits
        include       /etc/nginx/mime.types;
        default_type  application/octet-stream;
        log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
        access_log  /var/log/nginx/access.log  main;
        sendfile        on;

        # proxy behavior
        # https://www.nginx.com/resources/wiki/start/topics/examples/forwarded/
        proxy_redirect off;
        # this is VERY important to allow rev proxying to any port
        proxy_set_header   Host             $http_host;
        # make the "real ip"  the client address instead of the gateway
        proxy_set_header   X-Real-IP        $remote_addr;
        # append $remote_addr to any incoming X-Forwarded-For headers
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        # for large requests that dont fit into $proxy_buffers we need to govern the maximum size of the temp file that nginx will create
        proxy_max_temp_file_size 1024m;


        # give nginx some upstream servers to use for load balancing
        # you can define multiple of these in each upstream group and round robin or ip_hash (sticky ip) them
        upstream app1 {
                server 127.0.0.1:3001;
        }

        upstream app2 {
                server 127.0.0.1:3002;
        }

        ################
        #   0x8.host   #
        ################
        server {
                listen 80 default_server;
                listen 443 ssl http2 default_server;
                listen [::]:80 default_server;

                # we mount these certificates into the container and run certbot on the host
                ssl_certificate /etc/letsencrypt/live/0x8.host/fullchain.pem;
                ssl_certificate_key /etc/letsencrypt/live/0x8.host/privkey.pem;

                ssl_protocols TLSv1.3;

                # we are going to share files from here
                root /usr/share/nginx/html;
                autoindex on;

                # Add index.php to the list if you are using PHP
                index index.html index.htm index.nginx-debian.html;

                # define the server name so nginx knows to route requests for this domain here
                server_name 0x8.host;

                # serve some example content on root
                location / {
                        root   /usr/share/nginx/html;
                        index  index.html index.htm;
                }

                # proxy requests to their own webserver instances
                location /app1 {
                        proxy_pass http://app1/;
                }

                location /app2 {
                        proxy_pass http://app2/;
                }
        }
}
```

Once you have finished creating all your files, build the container image with `docker build -t gateway .` while inside the directory with the dockerfile.
Your final folder structure should look similar to this.

```output
├── docker-compose.yaml
├── dockerfile
├── html
│   ├── favicon.ico
│   └── index.html
└── nginx.conf
```

### Putting it all together

Ok, lets run our two template services, start the gateway and check that everything works.

Build your gateway container

```output
docker build -t gateway .
```

Start your backend services.

```output
cd backendContainers
docker-compose up -d
```

Then start your gateway, start without -d for now to investigate any issues, 50x, 40x errors etc.

```output
docker-compose up
```

### Hurdles/Problems/gotchas

**Incorrect permissions on files** - Some common problems/hurdles i faced when running the project was either getting a 404 on the index, this was caused by a user permissions issue with the index.html file and possibly the directory permissions so be sure to check those.

**couldn't access backend services from gateway** - The gateway couldn't access backend service containers on localhost, this was caused because the gateway's localhost is not the hosts localhost, therefore the services were inaccessible through gateway specifying 127.0.0.1.

## Debugging

### Disable HTTPS redirects

Heres the situation. I have 2 server blocks. Server block `roaming.host` is listening on port 443 for HTTPS traffic and server block `api.roaming.host` is listening on port 80 for HTTP traffic.

```none
server {
	listen 443 ssl;
	listen [::]:443 ssl;
	server_name roaming.host;

#	...code
}

server {
	listen 80;
	listen [::]:80;
	server_name api.roaming.host;

#	...code
}
```

What would happen if i tried to access `https://api.roaming.host` (on https/443)?

Nginx has documentation about how a process is requested [here](https://nginx.org/en/docs/http/request_processing.html), and in this situation your request to api.roaming.host will actually be passed to roaming.host instead by default. This is because of the order that nginx uses to determine the server block to pass the request to. In this case the given route starts with https so roaming.host is considered above api.roaming.host because the port matches (listen: 80 or listen: 443) outranks the importance of the server_name.

To fix this and redirect https requests to api.roaming.host which is not configured to work on https you should create a fallback server. In fact while you are at it create two to take care of the reverse situation (:80 request to a site that should be :443 getting redirected back to the :80 site).

These server blocks are hit as a last resort when nginx cant match the request to any other reasonable server in the config.

Ie when a request comes in for api.roaming.host:443 and api.roaming.host can only handle api.roaming.host:80 the default_server block is called instead of roaming.host:443

```none
# default server blocks
when a nginx
server {
	server_name roaming.host;
#	...
}

server {
	server_name api.roaming.host;
#	...
}

# Default server to prevent invalid ports
# ...being switched to sites that dont match that port
# This works because default_server is called as a last resort
server {
	listen 443 default_server;
	return 403;
}

# Another default server for the reverse
# ...(requests to :80 that should go to :443)
server {
	listen 80 default_server;
	return 403;
}
```

## Nginx Configuration

### HTTP2

WIP

### TLS 1.2 & TLS 1.3

WIP
