# MongoDB

## Install 
### Arch
MongoDB happens to be in the AUR as a prebuilt binary. This makes our job way easier. Just install with your favorite AUR helper/manager:
```
yay -S mongodb-bin
```
Optionally (and highly recommended) is to install the mongodb-compass app that provides a GUI for the mongo shell.
```
yay -S mongodb-compass
```

### Debian 10
Use the [official documentation](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-debian/).

#### Fixing Status Code 14
You may encounter an error when starting mongod (code=exited, status=14).
1. Check the permissions on /var/lib/mongodb and /tmp/mongodb-27017.sock
```
chown -R mongodb:mongodb /var/lib/mongodb
chown mongodb:mongodb /tmp/mongodb-27017.sock
```
2. Make sure your system time is correct, see my [Debugging notes](https://rolandw.dev/Notes/Linux/Debugging/) to set the correct time.

## Creating a User

### Admin User
Create a user in the **admin** database
```javascript
> use admin

> db.createUser(
	{
		user: "admin",
		pwd: "rhinos",
		roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
	}
)
```

### Non Privileged Users
```javascript
> use testdb

> db.createUser({
    user: 'roland',
    pwd: 'myPassword',
    roles: [{ role: 'readWrite', db:'Put_Your_Database_Name_Here'}]
})
```

```
db.createUser({ user: "roland", pwd: "rhinos", roles: [ "readWrite", "dbAdmin" ]})
```

## Connecting to MongoDB Remotely
Client computer: 60.224.98.212
mongoDB server: 139.180.169.10

### Setting up Mongo on the Server
1. Create a non privileged user
2. Enable authorization
```
# /etc/mongod.conf
security:
  authorization: 'enabled'
```
3. rebind localhost (172.0.0.1) to all addresses (0.0.0.0)
```
# network interfaces
net:
  port: 27017
  bindIp: 0.0.0.0
```
4. Open a port on your server for 27017
```
sudo ufw allow from 60.224.98.212 to any port 27017
sudo ufw reload
```
### Connecting to Remote Mongo Server
Connect to the remote mongo server
```
mongo -u roland -p myPassword 139.180.169.10/testdb
```
You can also connect using
```
mongo mongodb://roland:rhinos@139.180.169.10:27017/testdb
```


## Mongo Shell Commands
Commands can be found on the [docs](https://docs.mongodb.com/manual/reference/method/)

### Switching databases
Switch to a database: `use myDatabase`

### Get all the users of a database
1. Switch to the database
2. Run `db.getUsers()`

### Get the name of the current database
1. Switch to the database
2. Run `db.getName()`
3. Or run `db`

### List all databases
As an admin account with *userAdminAnyDatabase*
1. Run `db.adminCommand( { listDatabases: 1 } )`

As an unprivileged account
1. Run `show dbs`

### List all collections
1. Switch to the database
2. Run `db.getCollectionInfos()`
3. You can get just the array of collections with `db.getCollectionNames()`