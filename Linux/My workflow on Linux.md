# Workflow

## MPV

### MPV Scripting

I use these scripts for MPV

#### Delete File

[Source](https://github.com/zenyd/mpv-scripts).

`ctrl + DEL`: mark/unmark file to be deleted

#### Locate File

[Source](https://github.com/nimatrueway/mpv-locatefile-lua-script).

Copy the `locate-current-file.lua` script to *~/.config/mpv/scripts*.\
Create a key bind in *~/.config/mpv/input.conf*, for example `Alt+f script_message locate-current-file`.

#### MPV-Splicer

[Source](https://github.com/pvpscript/mpv-video-splice).

* `Alt+t` Start slice at this timestamp.
* `Alt+t` Finish a slice at this timestamp.
* `Alt+c` Export these slices.

Other stuff:

* `Alt+p` Print the number of slices.
* `ALt+d` Enter slice deletion mode, then hold `Alt+NUM` to delete that slice.

#### Rubber band helper

[Source](https://github.com/jgreco/mpv-scripts/).

Minimize unintelligible voices at high playback speeds at the cost of CPU.

#### Control and Redo playback

[Source](https://github.com/Eisa01/mpv-scripts).

Seek previous positions in a video that have been visited by scrubbing. Use `Ctrl+z` and `Ctrl+y`, acts as you would expect.

#### Auto save progress

Store a history of videos in a *"watch later"* style playlist and automatically resume where you left off.

#### Recent

[Source](https://github.com/hacel/mpv-scripts).

Store a log of recently watched videos and select them for playback. Open the recently played menu with the **`** key by default.

#### Next and Previous file

[Source](https://github.com/jonniek/mpv-nextfile).

`Shift+Left/Right` and move between files in the current directory.

#### Persist Properties

[Source](https://github.com/d87/mpv-persist-properties).

Keep selected values like volume between player sessions.

Modify `~/.config/mpv/script-opts/persist_properties.conf` with which properties you would like, for example.

```none
properties=volume,sub-scale
```

## Thunar

#### Trash Prompter

A small thunar user action that fixes the lack of delete prompt in thunar. A moment of silence for the lost files that have been put into the trash by accident.

Install `trash-cli` with `pacman -S trash-cli` or `apt install trash-cli`, and install zenity to prompt for deleting with a GUI. Then add this script to Thunar under *Edit->Configure Custom Actions*.

* Add new action.
* Name: "Delete Prompt" (any name will do).
* Command: `zenity --question --text="Delete %N?" && trash-put %F`.

## VIM

Come back soon. I dont know enough vim to make this yet.

### Using VIM in VSCode

I am still learning VIM. I am using the VIM [extension](https://github.com/VSCodeVim/Vim) for VSC.

### Getting vim copy paste

Luke smith did a great video about this [here](https://www.youtube.com/watch?v=E_rbfQqrm7g).

Vim uses registers to store text for later. registers can be named any letter.

1. Select a line of text in vim
2. Execute `"ay` -> Yank this to the a register ("a)
3. Likewise `"ap` will paste from a register

registers like a,b,c... come default in vim. However vim does not have access to the clipboard in Xorg without special compilation to support copy paste. The best way to avoid needing to recompile vim in a special way to support copying out of vim is to install [gvim](https://www.archlinux.org/packages/extra/x86_64/gvim/) which comes compiled with Xorg copy paste support (note that you never need to actually use gvim, it just enables it for regular vim).

Xorg and gvim use the + buffer ("+) to copy content to Xorgs clipboard

1. Select a line of text in vim
2. Execute `"+y` -> Yank this to the a register ("+)
3. Likewise `"+p` will paste from Xorgs clipboard

You can make this faster by creating key bind for it in your `.vimrc`

```none
" copy and paste in vim
" visual non recursive remap ctrl-c to yank to + register (xorg)
vnoremap <C-c> "+y
" Map ctrl-p to paste
map <C-v> "+p
```

## VSCode

### Misc Extensions

#### Better comments

aaron-bond.better-comments

> Colored comments, provides syntax for `// todo` and `// ! important`.

#### Bracket Pair Colorizer 2

coenraads.bracket-pair-colorizer-2

> Colored brackets.

#### Emoji

perkovec.emoji

>Provides ctrl+p -> insert emoji support.

#### Code Spell Checker

streetsidesoftware.code-spell-checker

> A basic spell checker that works well with camelCase code.

#### IntelliSense for CSS class names in HTML

zignd.html-css-class-completion

> Provides CSS class name completion for the HTML class attribute based on the definitions found in your workspace or external files referenced through the link element.

#### Live Server

ritwickdey.liveserver

> Locally hosts your HTML.

#### Markdown All in One

yzhang.markdown-all-in-one

> Essential for markdown

* Bolds and Italics
* list toggles (ctrl+c)
* ToC
* Other stuff

#### Markdown Lint

davidanson.vscode-markdownlint

> Linting for markdown

To ignore lint rules (such as MD010 no hard tabs in code fences) edit `~/.config/Code - OSS/User/settings.json` With the following.

```json
{
	"markdownlint.config": {
		"MD010": false
	}
}
```

#### React snippits

burkeholland.simple-react-snippets

> A list of react snippits

### Prettier

Prettier is a VSCode Extension for formatting code.

I am not sure if im a fan of it yet because its very opinionated and takes getting used to. Al alternate to check out is unibeautify, which still uses prettier to do some of the work (among other beautifiers) but may be more appropriate for non-js applications.

Install the package. Press ctrl+p and type `ext install esbenp.prettier-vscode`.

Next create your config file in a location on your computer. I put mine in my home or my \$XDG_CONFIG_HOME. You could also put it in your ~/.vscode\* folder.

#### Set default formatter

Next change your default formatter by going to editor -> default formatter or by searching for *default formatter*. Next using the dropdown change the default formatter to *esbenp.prettier-vscode*.

#### Set config path

Next press f1 and type settings. Then go to extensions -> prettier, or search for *prettier*. Then set **Prettier: Config Path** to your config location (eg /home/roland/.prettierrc).

#### Set ignore path

I have found the below instructions to NOT work but thats how its explained on the docs.

Next press f1 and type settings. Then go to extensions -> prettier, or search for *prettier*. Then set **Prettier: ignore Path** to your config location (eg /home/roland/.prettierignore).

Instead to actually do this (ie disable a language). Go to prettier.ignored languages in settings and add the language to the list. For example *markdown*

#### Set require config

Next search for *prettier.require* in the settings and check the *prettier: require config* box. This will make sure you are pointing to the default formatter (or any formatter).

### Vim commenting

Go to start of line and press `ctrl+v`. Then move the cursor up and down with arrow/hjkl. Then press `ctrl+shift+i` and type #. Then `Esc` and see all lines get commented.
