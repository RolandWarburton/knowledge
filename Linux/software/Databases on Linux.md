# Database stuff

### Setting up mongodb

I am aiming to use the most modern version of mongodb (currently 4.2). You need to follow weird instructions to install 4.2 because its not in the regular packages. Instructions [here](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/).

MongoDB config files can be found at `/etc/mongodb.conf`.

"By default, MongoDB runs using the mongodb user account." Though it has nologin privilages and so far i have found no good use for this information.

By default, MongoDBs database path (dbPath) is `/var/lib/mongodb`.

1. Install mongodb `sudo apt install mongodb`
2. Put mongo in your path. Check where mongo is installed with `which mongo`. Your path will usually be something like `PATH=/usr/bin/mongo:$PATH`
3. Unmask the mongod service because for some stupid reason its masked (masked is a more permanant version of disabled)
`sudo systemctl unmask mongod`
4. Start mongo with `mongod` command. you should see *waiting for connections on port 27017*
5. Enable mongo as a service with `sudo systemctl start mongodb` and `sudo systemctl enable mongodb`
If you type `mongod` again to try and start the service you will notice that it is already running. You can also confirm that mongo is running with `sudo systemctl status mongodb`.\
At this stage you can see mongodb, and its PID running with `sudo lsof -iTCP -sTCP:LISTEN`\
-iTCP = -i (select all of the) -TCP\
-sTCP:LISTEN = -s (filter) the -TCP ports that are is mode LISTEN

### Securing mongodb

its a good idea to require users to authenticate with the database.\
This also solves the **Access control is not enabled for the database** (happens on version 3.6) error.

1. create a privilaged superuser account to connect to the database with.

```javascript
use admin
db.createUser(
  {
    user: "superuser",
    pwd: "password",
    roles: [ "root" ]
  }
)
```

2. edit `/etc/mongod.conf` and enable `security.authorization`. This config files uses YAML markup.

```none
security:
   authorization: true
```

3. Then restart yout mongodb server and it will read the config file and enable authentication\
`sudo systemctl start mongod`

### Creating other types of mongodb users

Create an admin...

```javascript
use admin
db.createUser(
  {
    user: "roland",
    pwd: "password",
    roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
  }
)
```

Create a regular user...

```javascript
use any_db_name_here
db.createUser(
  {
    user: "roland",
    pwd: "password",
    roles: [ { role: "readWrite", db: "any_db_name_here" } ]
  }
)
```

You need to restart mongodb for it to gain access control and to verify that the error message is gone:\
`mongod --auth --port 27017 --dbpath /data/db`\
You can then connect to the database as that user with:\
`mongo --port 27017 -u "myUserAdmin" -p "abc123" --authenticationDatabase "admin"`
