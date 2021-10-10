# Dockerode Cookbook

Dockerode examples and recipes for automating [Docker](https://www.docker.com/) through its [SDK](https://docs.docker.com/engine/api/v1.37) with the [Dockerode API library](https://github.com/apocas/dockerode).

## Creating containers

Using typescript you can get intelisense for the [Container](https://docs.docker.com/engine/api/v1.37/#operation/ContainerCreate) object.

```ts
const options: Docker.ContainerCreateOptions = {
    name: "temp", // Mandatory name
    Image: "busybox",     // Mandatory image
    AttachStderr: true, // Print errors
    AttachStdout: true, // Print output
    AttachStdin: false, // Accept input
    Tty: true, // Attach standard (above) streams to a TTY
    OpenStdin: true, // Enable stdin
    StdinOnce: false, // Close stdin after one attached client disconnects
    Cmd: ["/bin/sh"], // the command to run when the container starts up
};
```

Creating volume mounts.

```ts
const optionsVolume = {
    // import the previous options
    ...options,
    // configure the volume mounts
    HostConfig: {
        Binds: ["sourceVolume:/source"],
    },
}
```

Creating port maps.

```ts
const optionsVolume = {
    // import the previous options
    ...options,
    // configure the port mapping
    HostConfig: {
        PortBindings: {
            "80/tcp": [{
                HostPort: "8080",
            }],
            "443/tcp": [{
                HostPort: "8443",
            }],
            "22": [{
                HostPort: "22",
            }],
        },
    },
}
```

## Exec a command in a container

Exec a command in a container and the log that to stdout on the host.

```ts
import Docker from "dockerode";

const docker = new Docker({ socketPath: "/var/run/docker.sock" });

const options: Docker.ContainerCreateOptions = {
    name: "temp",
    Image: "busybox",
    WorkingDir: "/source",
    Env: ["PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"],
    AttachStderr: true,
    AttachStdin: false,
    AttachStdout: true,
    Tty: true,
    OpenStdin: true,
    StdinOnce: false,
    Cmd: ["/bin/sh"],
    HostConfig: {
        Binds: ["sourceVolume:/source"],
    },
};

// create a container
const c = await docker.createContainer(options);
console.log(`Created ${c.id}`);

// start the container
await c.start();
console.log(`Started ${c.id}`);

// create an exec command to run inside the container
const exec = await c.exec({ Cmd: ["date"], AttachStdout: true, AttachStdin: false });
const out = await exec.start({ hijack: false, stdin: false, Tty: true });

// wait for data then print it
out.on("data", (data) => {
    console.log(data.toString());
});

out.on("close", async () => {
    console.log("stopping container");
    await c.stop();
    await c.remove({ force: false });
    console.log(`removed container ${c.id}`);
});
```
