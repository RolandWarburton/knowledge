# Docker Tidbits

#### Format docker ps output

Sometimes if you have lots of volumes, or ports mapped to your container the output from `docker ps` can get very cluttered, especially when viewing it on a vertical monitor (from experience).

Docker supports the use of `--format` to filter out which bits you want.

Here is a link to the [docs](https://build.rolandw.dev/build) explaining how this works, TLDR is using the Go template format.

Heres am example that i use to print short container names that i store as an alias in my `.bash_aliases` (or zsh, csh etc).

```none
alias "pspretty"="docker ps --format '{{.ID}}\t\t{{.Names}}\t\t{{.Status}}\t\t{{.Networks}}'"
```

## Docker compose

#### Daemonize docker-compose up

> You have started your docker image using `docker-compose up` and after doing some manual verification that its alive and working. You want to exit it and keep it running.

The solution to this is to "freeze" the process by pressing `ctrl+z`, this puts the application into the background and kicks you out of the docker-compose instance. Unfortunately it also stops the container, in this state it becomes unresponsive and will idle and consumer resources such as RAM.

Now run the `bg` command to bring the container out of its frozen state and place it in the background.

```output
[1]+ Stopped	docker-compose up
roland@server: bg
[1]+ docker-compose up &
```

Not its still technically not daemonized as its not flagged with -d. however it will continue to work like a normal container thats daemonized.

One weird downside i have noticed however is that this might break any logging you have.

Another (potential) downside to this is that output from the container will still be printed to stdout like you are still inside the container (ie you will still see container output).
