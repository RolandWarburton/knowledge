# Nginx

## Server Blocks

Server blocks are used to define different websites on the same server.

```none
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

### Full Config Example

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

### Debugging

#### Disable HTTPS redirects

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
