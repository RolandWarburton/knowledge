# ZSH

### What is ZSH

[ZSH](https://en.wikipedia.org/wiki/Z_shell) is an alternative shell to bash. Its very similar in syntax but comes with extra features that make it pretty neat.

### Setting up ZSH

1. Install using your distros package manager (pacman, apt, brew, or even WSL).
2. Set up a location to put your config files by exporting `$ZDOTDIR` through `~/.zshenv` in your home directory or /etc/zsh/zshenv.

```bash
# Make the new location for config files
mkdir -p ~/.config/zsh
# Tell ZSH to load configs from the new ZDOTDIR
sudo echo export ZDOTDIR=$HOME/.config/zsh >> /etc/zsh/zshenv
# Create config files in their new home
touch ~/.config/zsh/.zsh_aliases
touch ~/.config/zsh/.zshrc
# Or move them
cp ~/.zsh_aliases ~/.zshrc ~/.config/zsh
```

3. Change your shell

```bash
# Change the shell for the current user
chsh -s $(which zsh)
# Change the shell for root
sudo chsh -s $(which zsh)
```

4. Set your .zshenv

Tell zsh where to look per account by setting the .zshenv inside $HOME/.zshenv

```none
echo "EXPORT ZDOTDIR=$HOME/.config/zsh" >> $HOME/.zshenv
```

5. Start configuring!

### Setting up fpath

*fpath* is like *$PATH* but just for *ZSH*, you can see your fpath by running `echo $fpath`.

```bash
# sample output of fpath
/home/roland/.config/zsh/userFunctions /usr/local/share/zsh/site-functions /usr/share/zsh/site-functions /usr/share/zsh/functions/Calendar /usr/share/zsh/functions/Chpwd /usr/share/zsh/functions/Completion /usr/share/zsh/functions/Completion/Base /usr/share/zsh/functions/Completion/Linux /usr/share/zsh/functions/Completion/Unix /usr/share/zsh/functions/Completion/X /usr/share/zsh/functions/Completion/Zsh /usr/share/zsh/functions/Exceptions /usr/share/zsh/functions/Math /usr/share/zsh/functions/MIME /usr/share/zsh/functions/Misc /usr/share/zsh/functions/Newuser /usr/share/zsh/functions/Prompts /usr/share/zsh/functions/TCP /usr/share/zsh/functions/VCS_Info /usr/share/zsh/functions/VCS_Info/Backends /usr/share/zsh/functions/Zftp /usr/share/zsh/functions/Zle
```

In the default zsh config you get some useful functions for manipulating your fpath.

### fpath prepend

Prepend a directory to your fpath. You can now call functions from inside your fpath.

```bash
function fpath-prepend {
    [[ -d "$1" ]] && fpath=($1 $fpath)
}
# Prepend your own function directory to fpath
fpath-prepend "$HOME/.config/zsh/userFunctions"
```

### autoload functions

Say you have a directory thats in your fpath and you want to lazy load (autoload) the function into zsh (source the function only when needed). More info [here](https://unix.stackexchange.com/questions/33255/how-to-define-and-load-your-own-shell-function-in-zsh).

```none
.config/zsh/userFunctions/
├── bar
├── foo
└── hello
```

```bash
# .config/zsh/userFunctions/hello
#
#!/bin/sh
echo "hello!"
```

```bash
# .config/zsh/.zshrc
#
# With the -U flag, alias expansion is suppressed when the function is loaded.
# The flags -z and -k mark the function to be autoloaded using the zsh or ksh style.
autoload -Uz hello

# Ensure fpath does not contain duplicates
typeset -gU fpath
```

```bash
# now inside your terminal you can run 'which hello':
hello () {
	# undefined
	builtin autoload -XU
}
# now run the 'hello' command:
hello () {
	echo "hello!"
}
```

```bash
# .config/zsh/.zshrc
#
# Complete example
function fpath-prepend {
    [[ -d "$1" ]] && fpath=($1 $fpath)
}
fpath-prepend "$HOME/.config/zsh/userFunctions"
autoload -Uz hello
typeset -gU fpath
```

## ZLE - zsh line editor

For most of this information: [source](https://sgeb.io/posts/2014/04/zsh-zle-custom-widgets/).

The zsh line editor, aka "the command prompt", is the bridge between you and zsh.

### ZLE keymap modes

The zsh line editor comes with the concept of keymaps, Keymaps are a collection of keybindings.

When a new keymap is chosen, all keyboard shortcuts are replaced with the ones defined in that new keymap.

The following keymaps are set up by default in zsh:

* EMACS emulation mode
* viins - vi in insert mode
* vicmd - vi in command mode
* isearch - incremental search mode
* command - command reading mode
* .safe - keymap fallback

The default mode is in fact emacs, though most users dont use many of these emac style keybinds ( with exceptions like ctrl+u to clear line).

Some other emacs style keybinds worth trying are `alt+b`, `alt+f`, `ctrl+a`, `ctrl+e`, and `ctrl+w`.

`viins` and `vicmd` are the vim keymodes for fancier line editing. Using `alt+i` will place you in viins mode. The same rules apply to vicmd (`alt+c`).

Once in a particular mode, for example vicmd, normal vim keys will work as you expect. For example navigating backwards and forwards with `b` and `e`. for example While in viins you could press `alt+b` to go to vivmd and go backwards a word, then press `shift+a` to go back to the end of the line and automatically be placed back into viins.

### ZLE vim program line editor

When you have a large command you can use the entire vim program to edit it fullscreen using a keybind.

To do this you need to map the following in your zshrc.

```none
autoload edit-command-line; zle -N edit-command-line
bindkey -M vicmd v edit-command-line
```

Now `alt+v` will open vim to edit your command on.

