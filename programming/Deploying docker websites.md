# Deploying docker websites

Docker is a new tool that i have pledged to look at personally to better understand containers and virtualization. Recently one of my projects posed a good application of docker and probably in hindsight created more problems than it should have, however i am feeling much better having mastered the basics of using docker with website operations.

This post coverers my process of setting up **docker-letsencrypt-nginx-proxy-companion** along with a node application such as an API or other micro service.

![docker-letsencrypt-nginx-proxy-companion-diagram.svg](/media/docker-letsencrypt-nginx-proxy-companion-diagram.svg)

### Resources

I Read the following stuff myself to understand how to structure the task.

1. [Server Wise](https://blog.ssdnodes.com/blog). - Hosting multiple SSL-enabled sites with Docker and Nginx. [link](https://blog.ssdnodes.com/blog/host-multiple-ssl-websites-docker-nginx/).
2. [nginx-proxy](https://github.com/nginx-proxy). - docker letsencrypt nginx proxy companion README.md [link](https://github.com/nginx-proxy/docker-letsencrypt-nginx-proxy-companion).
3. [Wes Doyle](https://www.youtube.com/channel/UCfniixfhHqpIGbU7z2JCNJw). - Introduction to docker and nginx reverse proxy. [link](https://www.youtube.com/watch?v=hxngRDmHTM0).

## Method

First ill go over a high level overview of what we will be building in this example tutorial.

```output
├── app
│   ├── website
│   │   ├── app.js
│   │   ├── package.json
│   │   └── package-lock.json
│   ├── docker-compose.yaml
│   ├── dockerfile
│   └── nginx.conf
└── nginx-proxy
    ├── docker-compose.yaml
    └── nginx.tmpl
```

The project will consist of two main things. The **Nginx Proxy** which is provided by the [Nginx Proxy](https://github.com/nginx-proxy) team who developed this container along with its companion letsencrypt companion to do two things.

1. **Nginx Proxy** is a reverse proxy container which takes a request from a client to a domain and routes it to its correct server block using nginx
2. **Nginx Proxy Companion** is an addition to the Nginx Proxy that monitors for newly added websites and automatically performs creation/renewal of Let's Encrypt (or other ACME CAs) using [simple_le](https://github.com/zenhack/simp_le).

This allows you (the web developer) to quickly spin up new docker websites and have SSL configured quickly and automatically, this lets you move websites between VPS' quickly, scale fast, monitor many websites at the same time using docker monitoring software, and best of all run multiple websites from the one host.

## Instructions

### Step 1 - Create proxy network

Create a network for your various containers to interact on. We will call this network **nginx-proxy**

```none
docker network create nginx-proxy
```

### Step 2 - Setting up the reverse proxy

Make a new directory to create your various proxy services in at `./nginx-proxy`.

```output
├── app
│   └── .
└── nginx-proxy <- We are working here
    └── .
```

Then create the docker-compose.yaml file to ./nginx-proxy.

```yaml
version: '3'

services:
  nginx:
    image: nginx:1.13.1
    container_name: nginx-proxy
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - conf:/etc/nginx/conf.d
      - vhost:/etc/nginx/vhost.d
      - html:/usr/share/nginx/html
      - certs:/etc/nginx/certs
    labels:
      - "com.github.jrcs.letsencrypt_nginx_proxy_companion.nginx_proxy=true"

  dockergen:
    image: jwilder/docker-gen:0.7.3
    container_name: nginx-proxy-gen
    depends_on:
      - nginx
    command: -notify-sighup nginx-proxy -watch -wait 5s:30s /etc/docker-gen/templates/nginx.tmpl /etc/nginx/conf.d/default.conf
    volumes:
      - conf:/etc/nginx/conf.d
      - vhost:/etc/nginx/vhost.d
      - html:/usr/share/nginx/html
      - certs:/etc/nginx/certs
      - /var/run/docker.sock:/tmp/docker.sock:ro
      - ./nginx.tmpl:/etc/docker-gen/templates/nginx.tmpl:ro

  letsencrypt:
    image: jrcs/letsencrypt-nginx-proxy-companion
    container_name: nginx-proxy-le
    depends_on:
      - nginx
      - dockergen
    environment:
      NGINX_PROXY_CONTAINER: nginx-proxy
      NGINX_DOCKER_GEN_CONTAINER: nginx-proxy-gen
    volumes:
      - conf:/etc/nginx/conf.d
      - vhost:/etc/nginx/vhost.d
      - html:/usr/share/nginx/html
      - certs:/etc/nginx/certs
      - /var/run/docker.sock:/var/run/docker.sock:ro

volumes:
  conf:
  vhost:
  html:
  certs:

# Do not forget to 'docker network create nginx-proxy' before launch, and to add '--network nginx-proxy' to proxied containers.

networks:
  default:
    external:
      name: nginx-proxy

```

The job of *nginx-proxy* is to take the encrypted request from the client and then pass it through the *webproxy* network to its correct location as seen in the diagram.

The job of its companion is to set up SSL for HTTPS only, these two services work together to manage and redirect traffic over the *nginx-proxy* network.

![media/webproxy-network.svg](/media/webproxy-network.svg)

The nginx-proxy and proxy-companion services can now be started using `docker-compose up -d`. Note that i PREFER using docker-compose where possible as i feel it makes services easier to understand, however the official docker-letsencrypt-nginx-proxy-companion repo provides a `start.sh` and `.env.sample` which you can use to run without needing docker-compose, though i did not use it for this experiment so YMMV.

Check your running processes and observe the newly started containers.

```none
docker ps
```

```output
CONTAINER ID        IMAGE                                    COMMAND                  CREATED             STATUS              PORTS                                      NAMES
f3f7a8fb1467        jrcs/letsencrypt-nginx-proxy-companion   "/bin/bash /app/entr…"   4 hours ago         Up 4 hours                                                     nginx-proxy-le
4f2467ce3063        jwilder/docker-gen:0.7.3                 "/usr/local/bin/dock…"   4 hours ago         Up 4 hours                                                     nginx-proxy-gen
4c71b79aa85d        nginx:1.13.1                             "nginx -g 'daemon of…"   4 hours ago         Up 4 hours          0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp   nginx-proxy
```

```none
docker network ls
```

```output
NETWORK ID          NAME                DRIVER              SCOPE
91c5bf994cd5        bridge              bridge              local
fd32bf6ba869        host                host                local
36d76f4cead6        nginx-proxy         bridge              local <- Interested in this one
7248359103b7        none                null                local
```

### Step 3 - Clone nginx template for docker-gen

Docker-gen is spun up using the above docker-compose file. The purpose of docker-gen is to monitor for new website containers which will trigger the proxy companion to perform new SSL challenges and set up SSL etc (this all happens automagically so no need to sweat).

Inside of the nginx-proxy directory, use the following curl command to copy the developer’s sample nginx.tmpl file to your VPS.

```none
curl https://raw.githubusercontent.com/jwilder/nginx-proxy/master/nginx.tmpl > nginx.tmpl
```

Thats it! nginx-proxy and its companion are all set up! you should restart your *nginx-proxy* containers by running `docker-compose down` from inside *./nginx-proxy*.

### Step 4 - Create an example website

In step 4 we will create a test site using **ExpressJS** and **NodeJS** to test the functionality of the reverse proxy and its proxy companion.

Start by creating a new directory outside of nginx-proxy called **app**

```output
├── app <- We are working here
│   └── .
└── nginx-proxy
    └── .
```

Next Create a nested directory for a vanilla node project, you will need npm installed for this. Install NPM with `apt install npm` on debian based systems, and `pacman -S npm` for arch based systems.

```output
├── app
│   ├── website <- We are working here
│   │   └── .
│   └── .
└── nginx-proxy
    └── .
```

```none
cd ./app/website
npm init -y
```

Install the express dependency.

```none
npm i express
```

Next create a small webserver using express. Create a file called *./app/website/app.js*

```js
const express = require("express");
const app = express();
const port = 3000;

app.get("/", (req, res) => res.send("Hello World!"));

app.listen(port, () =>
	console.log(`Example app listening at http://localhost:${port}`)
);

```

We should now have a complete example website inside *./app/website*. Your files should look like this.

```output
├── app
│   ├── website
│   │   ├── app.js
│   │   ├── package.json
│   │   └── package-lock.json
│   └── .
└── nginx-proxy
    └── .
```

### 5 - Add package.json scripts

When we create a docker container for this example website its important to be able to boil the website down into a npm script. Add the following commands to package.json to allow starting the server from npm scripts.

```json
{
	"name": "website",
	"version": "1.0.0",
	"description": "",
	"main": "app.js",
	"scripts": {
		"start": "node app.js"
	},
	"keywords": [],
	"author": "roland",
	"license": "ISC",
	"dependencies": {
		"express": "^4.17.1"
	}
}

```

### Step 6 - Test example website locally

Before you move on to dockerizing this example website make sure it works correctly by running it as a standalone app.

```none
npm run start
```

Test you can connect over localhost:80. If not you may either...

1. Have a conflicting port (check the error message)
2. Not have express installed (check package.json dependencies)
3. May be accessing it from outside the VPS network (move the ./website directory to your local machine to test the functionality would be the easiest way of testing, however be aware that there may still be a problem with the VPS itself)
4. Make sure NodeJS and NPM are installed for local testing

### Step 7 - Dockerizing the example website

Great! Finally onto the last stretch.

Create the following files in *./website* directory.

```none
touch ./website/docker-compose.yaml
touch ./website/dockerfile
touch ./app/nginx.conf
```

#### 7.1 Configure nginx.conf

Create a simple forwarding *server block* to forward traffic to your node app to serve instead of the default nginx static files.

```none
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
	worker_connections 1024;
}

http {
	server {
		server_name localhost;
		listen 3000 default_server;
		server_name DOMAIN.host localhost 127.0.0.1;

		location / {
			proxy_pass http://website3:3000;
			proxy_set_header X-Forward-For $remote_addr;
		}
	}
}
```

#### 7.2 Configure dockerfile

Next use the dockerfile below to get started configuring an image.

```dockerfile
FROM nginx:latest
COPY nginx.conf /etc/nginx/nginx.conf

FROM node:latest
WORKDIR /app
COPY website/package.json /app
RUN npm install
COPY ./website /app
EXPOSE 3000
CMD ["npm", "start"]
USER node
```

#### 7.3 Configure docker-compose

Lastly use this as your docker-compose

```yaml
version: "3"
services:
    website2:
        container_name: website
        environment:
            - VIRTUAL_HOST=DOMAIN.host
            - LETSENCRYPT_HOST=DOMAIN.host
            - LETSENCRYPT_EMAIL=warburtonroland@gmail.com
            - VIRTUAL_PORT=3000
            - VIRTUAL_NETWORK=proxy-network
        image: website
networks:
    default:
        external:
            name: nginx-proxy
```

### 8 - Running the example website

Build the website image before running by using the docker build command inside *./app*.

```none
docker build -t website .
```

Then run the example website from *./app*.

```none
docker-compose up
```

Now navigate to your website and pray that it all works.

Congrats!!!
