# OUTDATED

THIS PAGE IS OUTDATED. See `Debugging.md` for more recent revision.

## Remote Debugging Node Containers

It seems that i am destined for a never ending circle of using breakpoints for debugging, and then falling back to `console.log("got here")` depending on if my project decides to cooperate with VS Codes remote debugging features. This

The purpose of this note is to lay out some situations that ive faced when setting up remote breakpoints. This is something that is useful to have in your toolkit, especially when designing micro services and managing many small containerized or otherwise difficult to access projects.

## Debugging NodeJS inside docker

Before i found the solution to this i was using VS Codes "Attach to remote container" feature to imbed myself within the container to work. This however is not an ideal solution because i was missing all my VSC extensions. For the record though, here is how i was able to debug from within the container.

First install the "Remote - Containers" plugin from the extensions store. Then to start debugging from within the container simply type `> attach to remote container` in the `F1` console, and select the node process from within there.

Next in the settings menu search for `"debug javascript use preview"` and **enable** it. **THIS STEP IS IMPORTANT**.

Now that we have covered the "within the container" method. Lets now do a better approach. By exposing port `9229` to the host machine, VSC can attach to the debug node server from within the container. Add the "Docker: Attach to Node" configuration to your `launch.json` using intellisense, or copy paste from here.

```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "type": "node",
            "request": "attach",
            "name": "Docker: Attach to Node",
            "remoteRoot": "/usr/src/app",
            "port": 9229,
            "address": "localhost",
            "localRoot": "${workspaceFolder}/app",
        },
        {
            "type": "pwa-node",
            "request": "launch",
            "name": "Launch Program",
            "skipFiles": [
                "<node_internals>/**"
            ],
            "program": "${workspaceFolder}/app/server/app.js"
        }
    ]
}
```

Pay close attention to `remoteRoot` and `localRoot`. They are very important and can cause the `unbound breakpoint` error that has plagued me for longer then i am willing to admit.

### LocalRoot

Say you had the following project layout:

```none
ls /home/roland/projects/myProject
```

```output
├── app <- node is in here!
├── docker-compose.yaml
└── dockerfile
```

We need to reflect these changes in the `localRoot` by adding `/app` to our workspace like below.

```json
{
    "localRoot": "${workspaceFolder}/app",
}
```

### RemoteRoot

Say you had the following dockerfile.

```dockerfile
FROM node:14

# ...output omitted

# Set up
# DOCKER LIVES IN /usr/src/app THIS IS IMPORTANT
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app

# Make sure you expose ports for 9229 while you are at it
EXPOSE 9229

# ...output omitted
```

This means that we need to map the `remoteRoot` to `/usr/src/app` as explained in the above dockerfile.

```json
{
    "remoteRoot": "/usr/src/app",
}
```

### Docker compose

Next ensure that you are exposing the correct ports for docker through docker compose if you are using that. See the below examples.

1. docker-compose.yaml
2. dockerfile
3. package.json

```yaml
version: "3"
services:
    myservice:
        build: .
        container_name: myservice
        environment:
            - PORT=8081
        ports:
            - "8081:8081"
            - "9229:9229" # development - bind the node debug port for remote debugging
```

Finally expose your node or nodemon thread to the world! Obviously make sure you do this behind a firewall and in a network you trust, else provide a more restrictive address range.

```json
{
    "scripts": {
            "start": "node index.js",
            "start:debug": "node --inspect=0.0.0.0 index.js",
            "watch:debug": "nodemon --delay 500ms --inspect=0.0.0.0 index.js"
        },
}
```

Now pressing `F5` should connect you to the container.
