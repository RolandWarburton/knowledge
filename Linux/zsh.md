# ZSH

### What is ZSH
[ZSH](https://en.wikipedia.org/wiki/Z_shell) is an alternative shell to bash. Its very similar in syntax but comes with extra features that make it pretty neat.

### Setting up ZSH
1. Install using your distros package manager (pacman, apt, brew, or even WSL).
2. Set up a location to put your config files by exporting `$ZDOTDIR` through `~/.zshenv` in your home directory or /etc/zsh/zshenv.
```
! Make the new location for config files
mkdir -p ~/.config/zsh

! Tell ZSH to load configs from the new ZDOTDIR
sudo echo export ZDOTDIR=$HOME/.config/zsh >> /etc/zsh/zshenv

! Create config files in their new home
touch ~/.config/zsh/.zsh_aliases
touch ~/.config/zsh/.zshrc

! Or move them
cp ~/.zsh_aliases ~/.zshrc ~/.config/zsh
```
3. Change your shell
```
! Change the shell for the current user
chsh -s $(which zsh)

! Change the shell for root
sudo chsh -s $(which zsh)
```
3. Start configuring!

### Setting up fpath
*fpath* is like *$PATH* but just for *ZSH*, you can see your fpath by running `echo $fpath`.

```
! echo $fpath

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

```
.config/zsh/userFunctions/
├── bar
├── foo
└── hello
```

```bash
# .config/zsh/userFunctions/hello

#!/bin/sh
echo "hello!"
```

```bash
# .config/zsh/.zshrc

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
# Complete example

function fpath-prepend {
    [[ -d "$1" ]] && fpath=($1 $fpath)
}

fpath-prepend "$HOME/.config/zsh/userFunctions"

autoload -Uz hello

typeset -gU fpath
```
