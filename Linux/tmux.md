# Tmux

## Configuring tmux

Heres a good config to get you started.

```none
# Rebind C-b to C-a to be like screen
unbind C-b
set -g prefix C-a

# 256 color support
set -g default-terminal "screen-256color"

# big scrollback energy
set -g history-limit 10000

# set pretty colors
#set -g status-bg black
#set -g status-fg white
set -g status-left ''
set -g status-right " #{session_name}"
setw -g window-status-current-style 'fg=colour28 bg=colour0 bold'

set -g pane-active-border-style 'bg=colour28'
set -g mouse off

# scrolling scrolls text up and not scroll through history
set -g mouse on
```

## Tmux basics

#### Creating sessions

Make a new session `tmux new -s mySession`. A session has its state preserved (through multiplexing).

Detach from a session with `ctrl+a d`.

Attach to a session with `tmux attach -t session_name`.

List the sessions running with `tmux ls`.

Rename a session with `tmux rename-session -t session_name new_name`. This can be done inside a session as well as outside.

Remove a session with `tmux kill-session -t session_name`.

#### Creating panes

Make a new horizontal pane `ctrl+a %`.
Switch panes with `ctrl+a <-` or ->

Make a new Vertical pane `ctrl+a "`.

#### Creating windows

Make a new window with `ctrl+b c`.
Switch windows using `ctrl+b 0` 1,2,3... etc.

Rename a window using `ctrl+b ,`.
