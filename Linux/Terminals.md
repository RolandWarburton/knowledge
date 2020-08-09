# Terminals

### Set default Lines and Columns

Export the `$Lines` and `$COLUMNS` variables to set it for terminals who respect them (all of them more or less).
The default on my computer was 50 lines * 112 columns

Terminals that respect `~/.Xresources` can also be configured via `TerminalName*geometry:  240x84`.
The defult for Xresources is 80 lines * 24 columns.

### Terminal key codes

Its useful to know the keycodes when you are mapping buttons to do things.

```none
showkey -a
```

Example:

```none
a 	97 0141 0x61
b 	98 0142 0x62
c 	99 0143 0x63
1 	49 0061 0x31
2 	50 0062 0x32
3 	51 0063 0x33
```

##### TermInfo files

`ls /usr/share/terminfo/x`\
List of supported terminals. use `echo $TERM` to what terminal the system thinks its using.\
You can export a new $TERM from the list above.

### Copy and paste in URxvt

install `urxvt-perls` then enable the clipboard pearl by adding `URxvt.perl-ext-common:  clipboard` to your ~/.Xresources (along with any other extensions you need)

## Setting up powerline and zsh

<!-- Install `powerline, powerline-fonts` and `zsh` package -->

make sure to include sudo
`sudo pip install powerline-shell`

then

```none
mkdir -p ~/.config/powerline-shell/
```

```none
powerline-shell --generate-config > ~/.config/powerline-shell/config.json
```

then edit like so referencing the [readme](https://github.com/b-ryan/powerline-shell)

```none
{
  "segments": [
    "virtual_env",
    "username",
    "hostname",
    "ssh",
    "cwd",
    "git",
    "hg",
    "jobs",
    "root"
  ],
  "cwd": {
        "max_dir_size": 3,
        "full_cwd": true
  }
}
```

Define a theme in `~/.config/powerline-shell/themes/`

```none
touch ~/.config/powerline-shell/themes/default.py
```

```python
from powerline_shell.themes.default import DefaultColor


class Color(DefaultColor):
    """Basic theme which only uses colors in 0-15 range"""
    USERNAME_FG = 0 # black
    USERNAME_BG = 7
    USERNAME_ROOT_BG = 1

    HOSTNAME_FG = 8
    HOSTNAME_BG = 7

    HOME_SPECIAL_DISPLAY = False
    PATH_BG = 8  # dark grey
    PATH_FG = 7  # light grey
    CWD_FG = 15  # white
    SEPARATOR_FG = 7

    READONLY_BG = 1
    READONLY_FG = 15

    REPO_CLEAN_BG = 2   # green
    REPO_CLEAN_FG = 0   # black
    REPO_DIRTY_BG = 1   # red
    REPO_DIRTY_FG = 15  # white

    JOBS_FG = 14
    JOBS_BG = 8

    CMD_PASSED_BG = 8
    CMD_PASSED_FG = 15
    CMD_FAILED_BG = 11
    CMD_FAILED_FG = 0

    SVN_CHANGES_BG = REPO_DIRTY_BG
    SVN_CHANGES_FG = REPO_DIRTY_FG

    VIRTUAL_ENV_BG = 2
    VIRTUAL_ENV_FG = 0

    AWS_PROFILE_FG = 14
    AWS_PROFILE_BG = 8

    TIME_FG = 8
    TIME_BG = 7
```

Then reference the theme inside `config.json`

```none
{
  "segments": [
    ...
  ],
  "theme": "~/.config/powerline-shell/themes/default.py"
}
```
