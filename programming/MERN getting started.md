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

#### Create a collection in your database
Once you have created the database (testdb). use mongodb-compass to create a collection inside the database (testCollection).

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

#### Create a Folder Structure
The structure of the projects front end should look like this
```
├── templates
│   └── template.html
├── webpack
│   ├── plugins.js
│   ├── modules.js
│   ├── paths.js
│   └── common.js
├── src
│   ├── styles
│   ├── components
│   └── index.js
├── routes
│   └── tracks.js
├── webpack.config.js
├── package.json
└── package-lock.json
```

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
The projects back end should look like this
```
├── routes
│   └── tracks.js
├── database
│   └── index.js
├── controllers
│   └── tracks.js
├── models
│   └── Tracks.js
├── api
│   └── index.js
├── server.js
├── package.json
└── package-lock.json
```

* A form from the front end submits data to the /api/track endpoint.
* Express picks up the submission and uses body-parser to extract the body of the text
* It then uses routes/tracks.js to find the correct controller to use (for example TrackController.createTrack). The route also passes the (req, res) parameters to createTrack
* TrackController.createTrack then uses the schema in models/Tracks to create a POST request to the database using mongoose


#### Serving React With Express (server.js)
1. Connecting middleware
   1. CORS and Bodyparser middleware. [reference this](https://expressjs.com/en/resources/middleware/cors.html) and [this](https://www.npmjs.com/package/body-parser). In most most normal situations body parser is parsing json, so call the middleware with `app.use(bodyParser.json())`.
      1. If you need to POST files (like images). The bodyparser middleware needs to be set to `app.use(bodyParser.urlencoded({ extended: false }))` to parse x-www-form-urlencoded. See reference [here](https://dev.to/getd/x-www-form-urlencoded-or-form-data-explained-in-2-mins-5hk6).
   2. Serving the static files ([reference this](https://expressjs.com/en/starter/static-files.html))

#### Connect the server to mongo (database)
To make the backend more readable and modular. We will create a separate directory called **database** to store the connection code to your server. This helps make your application more secure in case you eccidently exposed your server code on dist but also makes the server.js less cluttered.

Start by importing the database connection in server.js
```javascript
const db = require('./database')
```

Next write your connection
```javascript
// database/index.js

const mongoose = require('mongoose');
require('dotenv/config');

// connect to the database
mongoose
	.connect(process.env.DB_CONNECTION, {
		useNewUrlParser: true,
		useUnifiedTopology: true
	})
	.catch(err => {
		console.log(err)
	});

const connection = mongoose.connection;

connection.once('open', function() {
    console.log("MongoDB database connection established successfully");
})

module.exports = connection
```

#### Create a router to route axios http requests (routes)
Like the database connection. Routes can also be modularized to make the back end nicer to read and handle more routes in the future.
1. Create the routes/ directory
2. I put my tracks.js in routes/ and required it in my server.js

routes/tracks.js is going to be my api endpoint for axios so i require the routes in my server.js like so
```javascript
app.use('/api', tracks);
```

When i want to create a new route in routes/tracks the path will be relative to `/api`.

the routes/tracks.js file imports a TrackController. Each endpoint will call a function from the controller which will use mongoose to POST/GET/DELETE from the database.
```javascript
const express = require('express');
const TrackController = require('../controllers/tracks')
const router = express.Router();

// should return all tracks
router.get('/tracks', TrackController.getTracks)

// return a track by its ID
router.get('/track/:id', TrackController.getTrackById)

// delete a track by its ID
router.delete('/track/:id', TrackController.deleteTrack)

module.exports = router
```

##### What is the purpose of axios if we use mongoose to put data into the database?
Currently none.
The way the application/back end is set up right now does not use axios to do anything yet.

#### Create the a mongoose schema to use with axios (models)
In /models/Tracks.js create a schema which is like a template to construct json objects to submit to the server
```javascript
const mongoose = require('mongoose')

const Track = mongoose.Schema({
	title: {
		type: String,
		require: true
	},
	desc: {
		type: String,
		require: true
	}

}, { collection: 'testCollection' })

module.exports = mongoose.model('Track', Track)
```

#### Create controllers for POST, GET etc... (controllers)
In controllers/tracks.js you can create  functions for each action on the database. For example, POSTing, GETting, and DELETE(ing) data.

Make sure to import the mongoose schema to use to create a mongo object
```javascript
const Track = require('../models/Tracks')

// now you can create a mongo object to submit to the database with 
const track = new Track(body)
```

When you export your controllers use `module.exports` and export it as an array.
```javascript
module.exports = {
	createTrack,
	deleteTrack,
	getTracks,
	getTrackById
}
```

#### Generic POST
```javascript
createTrack = async (req, res) => {
	const body = req.body
	console.log("parsing")
	console.log(body)

	if (!body) {
		return res.status(400).json({
			success: false,
			error: 'nothing in body'
		})
	}

	const track = new Track(body)

	if (!track) {
		return res.status(400).json({ success: false, error: err })
	}

	track
		.save()
		.then(() => {
			return res.status(201).json({
				success: true,
				id: track.id,
				desc: track.desc,
				message: "success!"
			})
		})
		.catch((err) => {
			return res.status(400).json({
				success: false,
				err
			})
		})
}
```

#### Generic GET all
```javascript
getTracks = async (req, res) => {
	await Track
		.find({}, (err, tracks) => {
			if (err) {
				return res.status(400).json({ success: false, error: err })
			}
			if (!tracks.length) {
				return res
					.status(404)
					.json({ success: false, error: `Tracks not found` })
			}
			return res.status(200).json({ success: true, data: tracks })
		})
		.catch(err => console.log(err))
}
```

#### Generic GET by ID
```javascript
getTrackById = async (req, res) => {
	await Track
		.findOne({ title: req.params.id }, (err, track) => {
			if (err) {
				return res.status(400).json({ success: false, error: err })
			}

			if (!track) {
				return res
					.status(404)
					.json({ success: false, error: `Track not found` })
			}
			return res.status(200).json({ success: true, data: track })
		})
		.catch((err) => console.log(err))
}
```

#### Generic DELETE by ID
```javascript
deleteTrack = async (req, res) => {
	await Track
		.findOneAndDelete({ title: req.params.id }, (err, track) => {
			if (err) {
				return res.status(400).json({ success: false, error: err })
			}

			if (!track) {
				return res
					.status(404)
					.json({ success: false, error: `track not found` })
			}
			return res.status(200).json({ success: true, data: track })
		})
		.catch(err => console.log(err))
}
```

#### Implement the API with axios (api)
I am currently not using axios but i plan to eventually.

This should make writing the controllers easier ([reference](https://gist.github.com/PowellTravis/5e629532d7f48c7cfdf37e7e9201ccb7)).

#### Handling images
I have not finished researching this and this section will remain empty for some time (a long time) until i figure out how this works.

I need to read [this](https://dev.to/getd/x-www-form-urlencoded-or-form-data-explained-in-2-mins-5hk6) to learn about using x-www-form-urlencoded (for sending text/ascii) vs form-data for sending raw data (images etc) ([reference](https://stackoverflow.com/questions/26723467/what-is-the-difference-between-form-data-x-www-form-urlencoded-and-raw-in-the-p))

I should read about [form-data](https://www.npmjs.com/package/form-data) for parsing images through req.body to the controllers/tracks.js code to parse it and insert it into my database

Some other resources to read are this SO post [here](https://stackoverflow.com/questions/48863580/insert-data-into-mongodb-and-upload-image-issue-with-enctype-multipart-form-dat), and this blog post [here](https://bezkoder.com/node-js-upload-store-images-mongodb/)

One possible solution is to upload the file to a directory and then store a path to that file on the server in mongodb.

To do this you need:
1. `form-data` to construct a FormData object to store blob information about the image when you POST it to express
2. `formidable` to deconstruct the FormData object. You cannot use body-parser for this as it only works with x-www-form-urlencoded (ie. json) data only.

1. On `src/components/Home` in your `onsubmit()` function construct a new form for submission to your axios api function
```javascript
const form = new FormData()
		const input = document.querySelector('input[type=file]')
		form.append("image", input.files[0])
		form.append("title", "myTitle")
		form.append("desc", "myDesc")
		api.createTrack(form);
```
2. Your axios api then receives the FormData object as a single parameter and then passes it as an argument to the `/api/track` route as a POST request
3. Express picks up this POST request from axios and executes the `controllers/tracks.js` function which takes a `(req, res)` parameter and then imports formidable to deconstruct the `req` and extracts all the fields. It then modifies the `res` header with the now extracted json data.
```javascript
createTrack = (req, res) => {
	console.log("parsing the request through formidable")
	
	const form = formidable({ multiples: true });

	form.parse(req, (err, fields, files) => {
		if (err) {
			next(err);
			return;
		}
		res.json({ fields, files });
	});
}
```