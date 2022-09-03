# Child Process

Nodes [child process](https://nodejs.org/api/child_process.html#subprocesssendmessage-sendhandle-options-callback) allows for many ways to spawn and execute processes.

This note will cover use cases i have encountered and how child process is applied to them.

## Spawning and controlling a new shell

This is a sometimes useful trick to know, using `spawn` you can send and receive data as a stream between your node process and a child process.

In my use case, i am automating the following process.

1. Open a new terminal
2. Sourcing the file `~/.zplug/init.zsh`
3. Running `zplug install`

The first step is to use the `spawn` method to create a new shell process.

```js
import { spawn } from 'child_process';
const zshell = spawn('/usr/bin/zsh', { shell: false });
zshell.stdin.setDefaultEncoding('utf-8');
```

Great! As you can see we can call `zshell.stdin.<method>`, our first step is to set the encoding to utf-8. Next we can use `zshell.stdin.write` to send data to the child process. We expect these commands to print some data to stdout, so we will set up event listeners to capture that data.

```js
zshell.stdin.write('source ~/.zplug/init.zsh\n');
zshell.stdin.write('zplug install\n');

// when data arrives on stdout
zshell.stdout.on('data', (data) => {
  console.log(data.toString());
});

// when data arrives on stderr
zshell.stderr.on('data', (data) => {
  console.log(`stderr: ${data.toString()}`);
});

// this runs when the child process exits (but before it closes)
zshell.on('exit', (code) => {
  console.log(`child process exited with code ${code}`);
});

// this runs when the child process entirely closes
zshell.on('close', (code) => {
  console.log(`child process exited with code ${code}`);
});
```

One thing you might notice is that `zshell.stdin.write` is not going to wait between commands, it shoots off all the commands sequentially but instantly, for simple commands this is ok.

One problem that i have encountered is that complicated commands
