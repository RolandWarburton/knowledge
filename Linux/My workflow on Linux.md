# Workflow
### Using VIM in VSCode
I am still learning VIM. I am using the VIM [extension](https://github.com/VSCodeVim/Vim) for VSC.

# VIM Notes
Come back soon. I dont know enough vim to make this yet

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
```
" copy and paste in vim
" visual non recursive remap ctrl-c to yank to + register (xorg)
vnoremap <C-c> "+y
" Map ctrl-p to paste
map <C-v> "+p
```
