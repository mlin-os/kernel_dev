### Problems unsolved

#### Good plugins to try

[powerline](https://github.com/powerline/powerline)

[deoplete](https://github.com/Shougo/deoplete.nvim)

[python-mode](https://github.com/python-mode/python-mode)

#### How to config .vimrc?

May follow [this](https://github.com/NickolasHKraus/dotfiles/blob/master/.vimrc) to config my .vimrc.

#### How to solve problem related to ftplugin?

Here are some useful references:

https://stackoverflow.com/questions/158968/changing-vim-indentation-behavior-by-file-type

https://vi.stackexchange.com/questions/4/how-can-i-change-the-default-indentation-based-on-filetype

https://github.com/junegunn/vim-plug/issues/25

https://github.com/junegunn/vim-plug/issues/752

#### Learn vimscript

Here are some basic instuctions of [vimscript](https://learnvimscriptthehardway.stevelosh.com/).

### Tricks

1. How to reduce multiple blank lines to a single blank?

The [first way](https://unix.stackexchange.com/questions/12812/replacing-multiple-blank-lines-with-a-single-blank-line-in-vim-sed) is:
```Vim
:%!cat -s
```

The [second way](https://stackoverflow.com/questions/3032030/how-does-g-j-reduce-multiple-blank-lines-to-a-single-blank-work-in-vi) is:
```Vim
g/^$/,/./-j
```
