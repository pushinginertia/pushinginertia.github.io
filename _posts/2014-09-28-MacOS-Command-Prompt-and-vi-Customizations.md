---
layout: post
title: MacOS Command Prompt and vi Customizations
tags: [Configuration]
---

This is my custom command prompt configuration.

## Command Line

First, install the far better [iTerm](https://iterm2.com/).

Now edit `/etc/profile` and add the following. This can alternately be added to `~/.profile` to make the change local only to the current user.

```sh
# add vi to the command line for previous command lookups
set -o vi

# remove noise - show only the current directory on the command prompt
export PS1="\e[33m\w\e[37m $ "
```

## vi

vi can also be improved with syntax highlighting and some other usability improvements. The following is copied from [this source](http://geekology.co.za/article/2009/03/how-to-enable-syntax-highlighting-and-other-options-in-vim).

Edit `/usr/share/vim/vimrc` and add the following lines to the file.

```
set ai                  " auto indenting
set history=100         " keep 100 lines of history
set ruler               " show the cursor position
syntax on               " syntax highlighting
set hlsearch            " highlight the last searched term
filetype plugin on      " use the file type plugins

" When editing a file, always jump to the last cursor position
autocmd BufReadPost *
\ if ! exists("g:leave_my_cursor_position_alone") |
\ if line("'\"") > 0 && line ("'\"") <= line("$") |
\ exe "normal g'\"" |
\ endif |
\ endif
```
