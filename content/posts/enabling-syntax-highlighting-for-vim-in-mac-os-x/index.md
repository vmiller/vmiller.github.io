---
title: "Enabling Syntax Highlighting for Vim in Mac OS X"
date: 2014-01-03T07:14:07-05:00
draft: false
tags: [macOS]
---

Mac OS X ships with the vim editor, which supports syntax highlighting.  By default, however, syntax highlighting is not turned on.  Fortunately it is not hard to enable it.

Settings for vim are controlled by two files, one controlling settings globally and the other controlling settings for the user.  `/usr/share/vim/vimrc` is the file that will control the global settings.  changes made to this file will affect all users of the machine. On a new build of 10.9, here is what the file contains :

```
" Configuration file for vim
set modelines=0 " CVE-2007-2438
 
" Normally we use vim-extensions. If you want true vi-compatibility
" remove change the following statements
set nocompatible " Use Vim defaults instead of 100% vi compatibility
set backspace=2 " more powerful backspacing
 
" Don't write backup file if vim is being called by "crontab -e"
au BufWrite /private/tmp/crontab.* set nowritebackup
" Don't write backup file if vim is being called by "chpass"
au BufWrite /private/etc/pw.* set nowritebackup
```

The file for controlling vim settings for the user is `~/.vimrc` By default, this file does not exist. To turn on syntax highlighting, we can simply create a text file by that name and add this line :

```
syntax on
```

vim will now use syntax highlighting the next time a file is opened. But what if you don’t care for the default color scheme? We can set the color scheme by adding a second line to the .vimrc file like so :

```
syntax on
colo desert
```

My favorite color scheme is desert, I find it works nice with my preferred Terminal color scheme (Homebrew). To see what color schemes ship with Mac OS, look at `/usr/share/vim/vim73/colors` The .vim files in this directory are the color schemes. Just try different ones by changing the .vimrc file and find the one you like best.