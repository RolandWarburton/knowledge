# Remote Debugging Node Containers

It seems that i am destined for a never ending circle of using breakpoints for debugging, and then falling back to `console.log("got here")` depending on if my project decides to cooperate with VS Codes remote debugging features. This

The purpose of this note is to lay out some situations that ive faced when setting up remote breakpoints. This is something that is useful to have in your toolkit, especially when designing micro services and managing many small containerized or otherwise difficult to access projects.

## Debugging NodeJS inside docker

Before i found the solution to this i was using VS Codes "Attach to remote container" feature to imbed myself within the container to work. This however is not an ideal solution because i was missing all my VSC extensions. For the record though, here is how i was able to debug from within the container.

First install the "Remote - Containers" plugin from the extensions store. Then to start debugging from within the container simply type `> attach to remote container` in the `F1` console, and select the node process from within there.

Now that we have covered the "within the container" method. Lets now do a better approach. By exposing port `9229` to the host machine, VSC can attach to the debug node server from within the container. Add the "Docker: Attach to Node" configuration to your `launch.json` using intellisense, or copy paste from here.

Make sure that the `remoteRoot` is correctly set.

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
            "remoteRoot": "/usr/src/app"
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

Next in the `F1` menu search for `"debug javascript use preview"` and **enable** it.

Now pressing `F5` should connect you to the container.
