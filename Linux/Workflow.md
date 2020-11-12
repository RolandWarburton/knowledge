# Workflow

## MPV Scripting

I use these scripts for MPV

### Delete File

[Source](https://github.com/zenyd/mpv-scripts).

`ctrl + DEL`: mark/unmark file to be deleted

### Locate File

[Source](https://github.com/nimatrueway/mpv-locatefile-lua-script).

Copy the `locate-current-file.lua` script to *~/.config/mpv/scripts*.\
Create a key bind in *~/.config/mpv/input.conf*, for example `Alt+f script_message locate-current-file`.

### MPV-Splicer

[Source](https://github.com/pvpscript/mpv-video-splice).

* `Alt+t` Start slice at this timestamp.
* `Alt+t` Finish a slice at this timestamp.
* `Alt+c` Export these slices.

Other stuff:

* `Alt+p` Print the number of slices.
* `ALt+d` Enter slice deletion mode, then hold `Alt+NUM` to delete that slice.

### Rubber band helper

[Source](https://github.com/jgreco/mpv-scripts/).

Minimize unintelligible voices at high playback speeds at the cost of CPU.

### Control and Redo playback

[Source](https://github.com/Eisa01/mpv-scripts).

Seek previous positions in a video that have been visited by scrubbing. Use `Ctrl+z` and `Ctrl+y`, acts as you would expect.

### Auto save progress

Store a history of videos in a *"watch later"* style playlist and automatically resume where you left off.

### Recent

[Source](https://github.com/hacel/mpv-scripts).

Store a log of recently watched videos and select them for playback. Open the recently played menu with the **`** key by default.

### Next and Previous file

[Source](https://github.com/jonniek/mpv-nextfile).

`Shift+Left/Right` and move between files in the current directory.

### Persist Properties

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

### Vim splits

#### Splitting basics

**hsplit** with `:split` and **vsplit** `:vsplit`. You can shortcut this to `:sp` and `:vsp` respectively. A split can be created with different controls as well such as `10sp` to create a split thats 10 lines tall, or `sp file.txt` to split to another file.

Navigate between windows using `ctrl+w hjkl`, for speed you should remap these to use just `ctrl+hjkl`.

```vim
" Remap splits navigation to just CTRL + hjkl
nnoremap <C-h> <C-w>h
nnoremap <C-j> <C-w>j
nnoremap <C-k> <C-w>k
nnoremap <C-l> <C-w>l
```

By default when you create splits the default behavior creates splits on the wrong side (spawn on top or on right), change this to spawn below and to left with the following command for your ~/.vimrc config.

```none
" Window splits
set splitbelow splitright
```

#### Splitting manipulation

Resize window basics can be done with the `:resize` command, resize can take an absolute value or relative value, for example `:resize +5` or `:resize 5`. Vertical windows can be controlled with `:vertical resize <number>`.

Heres some keybinds for these using `Ctrl+left right up down`.

```none
" Adjust window sizes with keybind
noremap <silent> <C-Left> :vertical resize +3<CR>
noremap <silent> <C-Left> :vertical resize -3<CR>
noremap <silent> <C-Up> :resize +3<CR>
noremap <silent> <C-Down> :resize -3<CR>
```

Basically all window navigation or manipulation commands require the `Ctrl+w` prefix. Here are some useful examples.

* `Ctrl+w _` ^W then press underscore will maximize the focused window and reduce all other windows to 1 line in height.
* `Ctrl+w =` Not shift+= (just =) - Will equalize all panes evenly

To change the split from horizontal to vertical you can use the `TK` and `TH` commands i created below.

```none
" Go from horz to vert split
command TK wincmd K
" Go from vert horz to split
command TH wincmd H
```

To remove the pipe symbol on a vsplit use the following option. Note the whitespace after the \\.

```none
" Remove the pipe char on vsplits
set fillchars+=vert:\ 
```

#### Tabs

Now that ive covered splitting, next up is tabbing.

Create a tab with the `:tabnew` command.

Heres a key bind for it.

```none
" Create a new tab
noremap <silent> <Leader>c :tabnew<CR>
```

#### Vim multi cursors

[Source](https://github.com/mg979/vim-visual-multi).

* `ctrl+n` Select a work, repeat `ctrl+n` to find more of that word
* `ctrl+up/down` Select lines up or down
* `ctrl+n` then `q` to skip this selection and select the next one
* `ctrl+n` then `Q` to undo this selection and go back to the previous one (reverse of above)

#### Vim quote strings

[Source](https://github.com/tpope/vim-surround).

* With the cursor on a word **without** quotes run `ysiw"` iw is for "in-between word"
* With the cursor on a word **with** quotes run `cs'"` to change a single quote *'* to a double *"*
* Quote a whole line with `yss"`
* Delete a quotes word/line with `ds"`
* You can also write complex multi char quotes such as `ysiw<strong>`

### Vim Explorer

Use the `Explore` command to go into vims inbuilt file browser. In this mode you can rename, delete, and browse files and folders.

A great use of this is to use the Explore functionality when choosing a file to split into, use the `:split ./` and then *return* to bring up the file explorer instead of `:split <tab>` to tab through a list.

When scrolling files its important to remember how to scroll faster using the following keys. I only really use the half screen scrolls to reduce the chance of getting lost in a document.

* `ctrl+d` - Scroll down half screen
* `ctrl+u` - Scroll up half screen
* `ctrl+f` - Scroll 1 page down
* `ctrl+v` - Scroll 1 page up

### Opening files in vim - The find and buffer command

#### Opening files

To open a new file whilst in vim use the `:edit <filename>` command, `:e#` will take you back to the file you were editing before you jumpedj. To enable tab completion make sure you see the *wildmenu* code block in the next section (finding files).

#### Finding Files

using `:find <filename>` i am able to open a file while inside of vim, the find is not relative to the working directory so you should run find with `./` for the first directory you opened your file in.

For example to "set a workspace" run `vim .` in the directory you want to be the workspace root, this will bring up vims explorer and you can pick a file that way, or use the `:find filename` command straight from explorer.

```none
" Ignore node_modules when finding files and such
set wildignore+=**/node_modules/**

" Display all matching files when tab completing
set wildmenu
filetype plugin on
```

Once you have opened some files "into the buffer" - IE opened them at least once, you can use the `:b` command to search through the buffer for unique file names. For example:

```none
.
├── afilehere.txt
├── bfilehere.txt
└── init.vim
```

I run vim on `.` and then open afilehere, after that i then run `:find ./bfile<tab>` and open *bfilehere*, then i can navigate between them using `:b afile` and `:b bfile` to open these buffered files. The former find command also supports wildcards, so for example to find all text files `:find ./*.txt`.

Make sure to have `set wildmenu` and `filetype plugin on` enabled so that tab completion is displayed when running the find command. See below.

```none
" Display all matching files when tab completing
set wildmenu
filetype plugin on
```

To see the currently open files you can run the `:ls` command to see the buffered files. You can also navigate to these files from this menu too.

```output
:ls
  3 %a   "~/.config/nvim/init.vim"      line 1
  4 #    "./afilehere.txt"              line 1
Press ENTER or type command to continue
```

```none
:ls
  3 %a   "~/.config/nvim/init.vim"      line 1
  4 #    "./afilehere.txt"              line 1
:b 3
```

```none
:ls
  3 %a   "~/.config/nvim/init.vim"      line 1
  4 #    "./afilehere.txt"              line 1
:b afile
```

#### Gutentag - Ctags

Gutentag ([ludovicchabant/vim-gutentags](https://github.com/ludovicchabant/vim-gutentags)) is a plugin for generating Ctags and traversing through them.

Make sure you have installed `ctags` before installing gutentag with vim-plug or whatever you choose to use for vim plugin management.

Instruction to use

* `ctrl+]` Go to definition
* `ctrl+t` go back (pop tag off stack) - On a similar not `ctrl+o` will also go back but its manipulating the jump list, not the tag list.


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
