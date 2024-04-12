# Advanced Tmux

Spell book for tmux commands 🧙.

## Hooks

Hooks are a great way for tmux to send data to other applications via stdout.

You can also use tmux hooks to *trigger* commands to tmux itself.

### Hooks Basics

Hooks can be defined in this format.
By default a hook will be scoped to the current session it is invoked in.

```bash
tmux set-hook <hook-name> command
```

When we define a hook its typical to use the `-g` flag.
This will allow the hook to run across all sessions.

```bash
tmux set-hook -g <hook-name> command
```

If you do not want to target all sessions,
you can also target the current session using the `-t` flag.

```bash
tmux set-hook -t <session-name> <hook-name> command
```

To see all your hooks you can run the following as required.

```bash
tmux show-hooks -t # for this session
tmux show-hooks -g # for all sessions
tmux show-hooks -w # window hooks for this session
tmux show-hooks -gw # window hooks for all sessions
```

### Track Pane Visit History

Ensure focus events are enabled in your `.tmux.conf`

```bash
set -g focus-events on
```

Each tmux pane as an index number that we can use to send commands to.
To find the index of the current pane run.

```bash
tmux display-message -p "#{pane_index}"
```

Consider this command which creates an event that runs each time you focus a new pane.

Note you can also access many other variables listed [here](https://www.man7.org/linux/man-pages/man1/tmux.1.html).

```bash
tmux set-hook -g pane-focus-in "run-shell 'echo #{pane_index} > /home/roland/.tmux-pane-id'"
```

Lets improve this to keep only the last 10 pane_indexs in memory.

```bash
tmux set-hook pane-focus-in "run-shell '(P=\"$HOME/.tmux-pane-id\"; echo #{pane_index} >> $P && tail -n 10 $P > $P.tmp && mv $P.tmp $P)'"
```
