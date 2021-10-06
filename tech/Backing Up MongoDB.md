# Backing up MongoDB

To backup a mongo database you can use the `mongodump` tool provided by `mongo-tools` to get a bson backup of the database.

## Backing up

Mongo can be picky about versions of tools, and often the **mongo-tool** provided in distros like debian can be out of date, in my situation my mongodump tool was version 3.6 when my production database was version 4.0 so it caused an *Unrecognized field "snapshot"* error due to the version mismatch, the best way to make them play nicely is just to use the correct version in the first place.

To ensure that both are the same version, you can use this command and spawn the mongodump tool inside a docker container to pick which version you want to use for your own database.

In the below example i connect to my mongo database at 192.168.0.100 with the username blogwatcher and password pass123.

```none
docker run \
-it --name "mongodump" --rm \
-v /home/roland/backup:/data \
-w /data/ \
mongo:4.0 \
mongodump \
--uri="mongodb://blogwatcher:pass123@192.168.0.100:27017/?authSource=blogwatcher" \
--out="/data/backup"
```

* `-v /home/roland/backup:/data` Mount the backup directory on the host to /data
* `-w /data/` change the working directory to /data

Before running docker, make sure to set the permissions of your directory on the host to 777 to ensure it has write permissions.

This will output folders for you to restore from.

## Restoring from backup

Similar to backing up, we can restore from a docker container as well using a command similar to this.

In this example i backup to the same collection, because of this duplicate keys errors will happen and these records will be ignored and not overwritten, using `--drop` is one solution to allow for these restoring records to overwrite the existing ones, however this will also drop entire existing collection (if it exists) meaning that values in database B that are not in A will not be preserved.

A solution to the duplicate key problem is to note down all the duplicate key IDs are run `db.collection.remove({_id: ObjectId(...)})` on them with an editor or a script.

```none
docker run \
-it --name "mongodump" --rm \
-v /home/roland/backup:/data \
-w /data/ \
mongo:4.0 \
mongorestore \
--uri="mongodb://blogwatcher:rhinos@192.168.0.100:27017/?authSource=blogwatcher" \
--dir=/data/backup
```
