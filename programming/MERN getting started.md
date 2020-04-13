# Getting started with MERN

MERN is a full web stack that includes:
1. MongoDB
2. Express
3. React
4. NodeJS

## Pre setup
Make sure your environment is set up.

#### Install your node environment
Install nodejs, and node package manager
```
sudo pacman -S nodejs npm
```
Install node version manager and the latest version of node. More details on nvm [here](https://rolandw.dev/Notes/Linux/NodeonLinux/).
```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash
```

#### Install and set up your mongo environment
Theres 2 different ways for setting up mongo. One is for apt package managers and the other for pacman/arch based package managers.

**ONLY** the server that is serving your MERN app needs mongo-db shell installed. I have provided instructions for both Debian and Arch for installing the shell but i will be only needing the shell installed on Debian for this because it is going to be my server.

##### MongoDB install on arch
MongoDB happens to be in the AUR as a prebuilt binary. This makes our job way easier. Just install with your favorite AUR helper/manager:
```
yay -S mongodb-bin
```
Optionally (and highly recommended) is to install the mongodb-compass app that provides a GUI for the mongo shell.
```
yay -S mongodb-compass
```

##### MongoDB install on debian
Using the [mongo docs](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-debian/) You can copy paste your way to installation on either Debian 9 "Stretch", or Debian 10 "Buster".

You can also get the mongodb-compass software for debian but in this example i am going to use arch as my host so i dont need to install it here.

## Setting up MongoDB
This can be a complex process but i am going to cover only the minimal setup that you might want to use for experimenting with MERN and sacrificing easy access for relaxed security.

Further explanation on how to use mongodb will be after this section, like creating a collection and viewing the data within mongodb-compass.

My full notes on mongodb are [here](https://rolandw.dev/Notes/Linux/MongoDB/)

Once mongodb is installed you can manage the service with
```
# Start the service
sudo systemctl start mongod
# Stop the service
sudo systemctl stop mongod
# Get the status of the service
sudo systemctl status mongod
# Reload all services when you make changes to mongod configs
sudo systemctl daemon-reload
```

#### Create a default admin user
Switch to the admin table with `use admin` and create a user.
* This user is stored in the admin table. They have the role to administrate any table.
```javascript
db.createUser(
	{
		user: "admin",
		pwd: "rhinos",
		roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
	}
)
```
One line for copy paste:
```
db.createUser({user: "admin",pwd: "rhinos",roles: [ { role:"userAdminAnyDatabase", db: "admin" } ]})
```

#### Create a Non Privileged User
Create a new regular user for every database or project you are working on.
* Make sure to create the new user in the database you want them to administrate. They will inherit admin rights over that database with the *dbAdmin* and *readWrite* roles

Switch to the database you want to create. Eg. `use testdb`
```javascript
db.createUser(
	{ 
		user: "roland", 
		pwd: "rhinos", 
		roles: [ "readWrite", "dbAdmin" ]
	}
)
```

#### Authenticating remote connection to mongo
Usually when your app is running in production there is no need to allow connections from outside the local network to access the database. This is the default setting that mongo employs to keep the database secure.

Because we are testing and learning i am going to open the shell to allow remote connections from only my own IP address.

##### Checklist before enabling public connections
1. Make sure you have created a non privileged or admin user before this step
2. Make sure you have enabled authorization in `/etc/mongod.conf`

##### Setting up remote connections
Navigate to `/etc/mongod.conf` and enable authorization
```
# /etc/mongod.conf
security:
authorization: 'enabled'
```

Within the same config file bind all addresses (or if you like just your one address)
```
# /etc/mongod.conf
# network interfaces
net:
port: 27017
bindIp: 0.0.0.0
```

Mongo will now accept any connections from any IP address. Finally add your local machines IP to the firewall of the mongo server.
```
sudo ufw allow from 60.224.98.212 to any port 27017 &&
sudo ufw reload
```

#### Test connection with the mongo server
There are 3 ways of connecting to mongodb.

1. Shell connection using flags
```
mongo -u roland -p myPassword 139.180.169.10/testdb
```
2. Shell connection using a url 
```
mongo mongodb://roland:rhinos@139.180.169.10:27017/testdb
```
3. mongodb-compass connection using username and password. I find that this only works when using separate fields (click the top left button to switch to field entry rather than  URL entry)

## Creating the node project
The initial setup of the node project is very straightforward

#### Packages to install
##### Dev dependencies
1. `@babel/core` The core of babels environment
2. `@babel/plugin-transform-runtime` Polyfill Support for async functions on react components that are bundled into the browser
3. `@babel/preset-env` babels default environment
4. `@babel/preset-react` The default environment and polyfills for react
5. `babel-loader` Webpack needs this to handle babel
6. `concurrently` A tool for running webpack and express at the same time
7. `css-loader` Webpack needs this to handle css
8. `dotenv` Provide environment variables for API keys and other stuff
9. `html-webpack-plugin` For providing webpack with html templates
10. `jest` For testing
11. `node-sass` For compiling sass to css (used by webpack)
12. `nodemon` For monitoring and automatically restarting node processes
13. `react-helmet` For changing the head/meta tags of react rendered pages
14. `sass-loader` Webpack needs this to handle sass
15. `style-loader` Webpack needs this to handle css
16. `url-loader` Helps webpack handle png, jpg, and gif files
17. `webpack` Webpack itself
18. `webpack-cli` Webpack CLI commands
19. `webpack-merge` A webpack tool for creating modular webpack configurations

##### Dependencies
1. `axios` For performing GET and POST requests to mongo as promises on the server side
2. `body-parser` Exposes req.body to allow you to extract submitted values from forms. IE. Allows you to handle POST requests.
3. `cors` For making requests to outside resources
4. `express` The backend for handling form requests, serving react and handling form requests, and the connection to mongo
5. `mongoose` Nodes interface to mongo. Allows you to perform database actions
6. `react` The core react package
7. `react-dom` React dom
8. `react-router` A subset of react-router-dom. Exports the BrowserRouter, Switches etc
9. `react-router-dom` Extends reacts internal router (react-router). It also exports "dom aware" components such as `<Link />`
10. `react-table` For creating tables from data pulled in by axios as a promise and making it pretty (and in theory easier, but i dont understand the library that well).

## Creating the Front End
Come back to me. Not done yet :smile:

#### Building the webpack configuration
1. Common configuration
2. modules
3. plugins

#### Setting up Babel and Dotenv
1. babelrc (babel configuration)
   1. Your babelrc should have your two presets for babel to work and for it to correctly handle react
```javascript
{
	"presets": [
		"@babel/preset-react", "@babel/preset-env"
	],
	"plugins": ["@babel/plugin-transform-runtime"]
}
```
2. dotenv
   1. Your dotenv should store the connection to the database as a variable. 
```
DB_CONNECTION=mongodb://<username>:<password>@139.180.169.10:27017/testdb
```

## Creating the Back End
Come back to me. Not done yet :smile:

#### Serving React With Express
1. Connecting middleware
   1. CORS and Bodyparser middleware. [reference this](https://expressjs.com/en/resources/middleware/cors.html) and [this](https://www.npmjs.com/package/body-parser). In most most normal situations body parser is parsing json, so call the middleware with `app.use(bodyParser.json())`.
      1. If you need to POST files (like images). The bodyparser middleware needs to be set to `app.use(bodyParser.urlencoded({ extended: false }))` to parse x-www-form-urlencoded. See reference [here](https://dev.to/getd/x-www-form-urlencoded-or-form-data-explained-in-2-mins-5hk6).
   2. Serving the static files ([reference this](https://expressjs.com/en/starter/static-files.html))

#### Creating a router to route axios http requests
Come back to me. Not done yet :smile:

#### Creating the a mongoose schema to use with axios
Come back to me. Not done yet :smile:

#### Using axios to create controllers for POST, GET etc...
Come back to me. Not done yet :smile:

#### Implementing the API with axios
Come back to me. Not done yet :smile:
