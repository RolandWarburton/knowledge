# Boot MongoDB with JS Init Script

Take this MongoDB service.

```yaml
version: "3.1"
services:
    mongo:
        container_name: mongo_database
        build:
            context: .
            dockerfile: dockerfile
        environment:
            MONGO_INITDB_DATABASE: pet_database # this will be important later
            MONGO_INITDB_ROOT_USERNAME: admin
            MONGO_INITDB_ROOT_PASSWORD: password
        volumes:
            - database_volume:/data/db
        ports:
            - "27017:27017"
        expose:
            - 27017
```

We have not yet created a user to access the `pet_database` database. We cannot use the ROOT user becausethe root user does not / should not have access to tables other than the admin ones.

So we need to run some init script automatically to create our user. To do this its easiest to create a `dockerfile` and then run a custom image based on the `mongo:4.4` image.

```dockerfile
FROM mongo:4.4
COPY ./mongo-init.js /docker-entrypoint-initdb.d
RUN chown mongodb:mongodb /docker-entrypoint-initdb.d/mongo-init.js
```

The file permissions of the `mongo-init.js` on the host are `755` which seems to work fine. However we need to chown the file to the mongodb user inside the container to make sure it can be run.

```js
// mongo-init.js

print("============================================================================");
print("==================== RUNNING MONGO INIT-MONGO.JS SCRIPT ====================");
print("============================================================================");

db.createUser({
    user: "blogwatcher",
    pwd: "rhinos",
    roles: [
        {
            role: "dbOwner",
            db: "blogwatcher",
        },
    ],
});
```

The `db` in this script refers to the `$MONGO_INITDB_DATABASE` that we set in the compose environment variables.

If you need to create a whole bunch of databases you could write something like this.

```js
// mongo-init.js

print("============================================================================");
print("==================== RUNNING MONGO INIT-MONGO.JS SCRIPT ====================");
print("============================================================================");

const databases = ["blogwatcher", "test"];

for (let i = databases.length - 1; i >= 0; i--) {
    db = db.getSiblingDB(databases[i]);

    db.createUser({
        user: "roland",
        pwd: "rhinos",
        roles: ["readWrite"],
    });
}
```
