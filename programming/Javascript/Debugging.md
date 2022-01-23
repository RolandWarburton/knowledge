# Debugging Node

Debugging JS can be done using the command like like pdb for python.

Create a file called `test.js` to debug.

```js
function addTwo(n) {
  return n + 2;
}

const addN = (f, n) => f(n);

const result = addN(addTwo, 5);
console.log(result);
```

We can run this with node `node test.js`.

If we want to debug it we can use `node inspect test.js`

By default node will break on the first executable line, to avoid this run `NODE_INSPECT_RESUME_ON_START=1 node inspect test.js`

Lets modify the code to debug the addTwo function.

```js
function addTwo(n) {
  debugger;
  return n + 2;
}

const addN = (f, n) => f(n);

const result = addN(addTwo, 5);
console.log(result);
```

We can now inspect the state using `NODE_INSPECT_RESUME_ON_START=1 node inspect test.js`.

```output
❯ NODE_INSPECT_RESUME_ON_START=1 node inspect test.js
< Debugger listening on ws://127.0.0.1:9229/a5e45ca8-401b-41f7-8987-c6311883fe6e
< For help, see: https://nodejs.org/en/docs/inspector
<
connecting to 127.0.0.1:9229 ... ok
< Debugger attached.
<
break in test.js:2
  1 function addTwo(n) {
> 2   debugger;
  3   return n + 2;
  4 }
< Debugger attached.
<
debug>
```

## Debugging Modes

### Debug Mode

There are two main debug modes.

The first is `debug>` which is the default mode, this mode lets you step through code using...

```output
c    - contine
n    - next
s    - step in
o    - step out
bt   - backtrace
repl - enter a repl
```

### Repl Mode

If we want to inspect the value of a particular variable we can drop into `repl` mode.

Try printing out the value of `n`.

```output
...
debug> repl
Press Ctrl+C to leave debug repl
> console.log(n)
< 5
<
> undefined
< Debugger attached.
<
debug> c
< 7
<
< Waiting for the debugger to disconnect...
<
```

## Debugging With NDB

[ndb](https://www.npmjs.com/package/ndb) is a chrome debugger thats lets you do more when debugging.

The main advantage of ndb is that you can see the local scope, which is something nodes inspection cannot do.

```none
npm install  -g ndb
```

Then run `ndb test.js`. The chrome dev tools should open and you can get debugging.

## Debugging With The Built in Chrome Node Debugger

First make sure you have chrome installed for this. Then run `node --inspect-brk test2.js`.

The `--inspect-brk` will break on the first executable line. Using just `--inspect` will not.

Now open chrome, navigate to "chrome://inspect" and click the inspect button for the "remote target".

## Debugging Docker Containers

Using the same style of above, debugging docker containers can be done with the chrome node debugger as well.

Lets make a docker container for test2.js

```dockerfile
FROM node:16
COPY test2.js /test2.js
CMD node --inspect-brk=0.0.0.0:9229 /test2.js
```

```none
docker build -t debugging_node_test . && \
docker run --rm -p 9229:9229 --name debugging_node_test debugging_node_test
```

After running the container you should be able to see it in chroms [chrome://inspect#devices](chrome://inspect#devices) window.

## Debugging in VS Code

Outside of a container you can use a conventionally generated `launch.json` like this.

```json
{
  // Use IntelliSense to learn about possible attributes.
  // Hover to view descriptions of existing attributes.
  // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "type": "pwa-node",
      "request": "launch",
      "name": "Launch Program",
      "skipFiles": ["<node_internals>/**"],
      "program": "${workspaceFolder}/test2.js"
    }
  ]
}
```

Above will launch in VS Codes debug terminal, if you are like me and prefer to run in a separate terminal, then you can use the docker config below .

Within a docker container you can debug in VS Code using a `launch.json` like this.

```json
{
  // Use IntelliSense to learn about possible attributes.
  // Hover to view descriptions of existing attributes.
  // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
  "configurations": [
    {
      "name": "Docker: Attach to Node",
      "type": "node",
      "request": "attach",
      "port": 9229,
      // when running in dev, set the IP to this through the docker-compose.yaml file
      "address": "172.22.0.102",
      "localRoot": "${workspaceFolder}",
      "remoteRoot": "/usr/src/app",
      "protocol": "inspector"
    }
  ]
}
```

You can use the `--ip` flag to set the IP address of the container.

Or you can set the IP address in the docker-compose.yaml file.

```yaml
services:
  myservice:
    networks:
      gateway_network:
        ipv4_address: 172.22.0.102
```
