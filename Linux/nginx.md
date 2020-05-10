# Nginx

## Server Blocks
Server blocks are used to define different websites on the same server.

```
# Server blocks are located in nginx configs
# define a file fot each website
# If you are suing subdomains, they belong in one file along with the website itself

/etc/nginx/sites-available/
├── default
└── roaming.host

```

## Server block files
A server block may be wrapped in `http{...}` but depending on if you should wrap your website in http depends on how your `/var/www/*` is set up.

### Servers
**Only** wrap your servers in a http block if they are inside `/var/www/http/*`

I prefer to structure my websites from the www folder so for this example its not needed to wrap the server in http because its not inside the http directory.

```
/var/www
├── api.roaming.host # 'root' location for a server block
│   └── index.html
├── html
│   └── index.nginx-debian.html # default nginx server block
└── roaming.host
    └── index.html
```

```
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

```
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
```
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
```
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

```
/var/www
├── roaming.host # my main website
│   └── index.html
└── secret # super secret page outside of root
    └── index.html
```
You would like to link `/var/www/secret` to your main website when you navigate to `roaming.host/secret`

```
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
```
# WRONG
location = /secret {
		root /var/www/secret/;
}

# If i use the URL roaming.host/secret
# nginx will interpret it is /var/www/app/secret/secret 
# (/secret from the URL is APPENDED)
```

```
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

```
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

## Nginx Configuration

### HTTP2
WIP

### TLS 1.2 & TLS 1.3
WIP
