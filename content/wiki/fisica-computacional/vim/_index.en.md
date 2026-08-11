+++
title = '3. Vim'
date = '2026-01-10'
tags = ['vim', 'linux']
+++

Vi IMproved, a text editor for programmers.

There are other text editors such as `nano`, `neovim`, or `emacs`. They all
have different goals, but they serve the same basic purpose: editing text.

Vim comes with most Linux distributions, which is very convenient because it
is a text editor that may look simple at first, but it is very powerful once
we learn how to use it.

In most `vim` installations, the `vimtutor` tool comes with it. To use it, we
type `vimtutor` in the terminal and follow the steps to complete the different
tasks it presents.

Besides learning to use this program's shortcuts, we can also configure it to
our liking. Here is a simple configuration we can apply by writing it in the
`~/.config/vim/vimrc` file:

```vim {filename="~/.config/vim/vimrc"}
" When searching (with the '/' key), ignore upper and lower case.
set ignorecase

" If one letter in the search is uppercase, then the search becomes
" case-sensitive again.
set smartcase

" When searching, highlight the matches.
set hlsearch

" In case of crashes, accidental closes, or simultaneous edits,
" preserve edits in a separate swapfile so lost changes can be recovered.
set swapfile

" Persist the change log so we can consult it when opening
" a previously edited file and move back through edits
" even after closing the program.
set undofile

" Allow recursive search through a project's folders.
set path+=**

" Enable the mouse for navigation and selection.
set mouse=a

" Extra information in the status line. To see exactly how it works,
" just like with any of these options, we can run this inside Vim:
" :help showcmd
" and, like with any Vim buffer, we can quit with :q
set showcmd

" Allow switching between buffers without saving changes
set hidden

" Show certain characters such as:
"   - Extra spaces at the end of the line
"   - Tabs showing indentation
"   ...among others
" This makes unnecessary characters, such as trailing spaces, visible.
" It also helps visualize indentation levels.
set list
```

To see other configuration options, we can consult `:help vimrc`. Also, the
internet is a very good resource for customizing our configuration in detail.
For example:

{{< youtube XA2WjJbmmoM >}}
