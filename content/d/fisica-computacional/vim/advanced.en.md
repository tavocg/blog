+++
title = 'Advanced Configuration'
date = '2026-03-20'
tags = ['vim', 'linux']
+++

## Multiple Configuration Files

Keeping all our configuration in a single file can be tedious, but we can take
advantage of the fact that Vim's configuration language is a full programming
language called Vimscript.

Our goal is to have a folder in the same directory as our `vimrc`, called
`conf.d/`. In this folder we can add separate configuration files that load
every time we run Vim. We will start by creating an isolated folder so we do
not alter the existing configuration.

```sh
mkdir -p ~/.config/myvim
touch ~/.config/myvim/vimrc
```

To use this configuration, we can call Vim like this:

```sh
export VIMINIT="source ~/.config/myvim/vimrc"
vim
```

> [!NOTE]
> You can define this environment variable in `~/.bashrc` to persist it across
> sessions and use this configuration by default every time you start Vim.

Now that we have a way to access an isolated, empty base configuration, we can
start by writing the following:

```vim {filename="~/.config/myvim/vimrc"}
for file in split(glob(expand('<sfile>:p:h') . '/conf.d/*.vim'), '\n')
  execute 'source' fnameescape(file)
endfor
```

This directive says that, for each `conf.d/*.vim` file, where `conf.d` is a
folder relative to our `vimrc` and `*.vim` is a wildcard that selects any file
with the `.vim` suffix, Vim will include it in the current execution.
So we create the configuration folder to keep each setting in its own file.

Let's test it:

- Create `~/.config/myvim/conf.d/custom.vim`

```sh
mkdir -p ~/.config/myvim/conf.d
touch ~/.config/myvim/conf.d/custom.vim
```

- Define a global variable

```sh
vim ~/.config/myvim/conf.d/custom.vim
```

```vim {filename="~/.config/myvim/conf.d/custom.vim"}
let g:loaded_custom = 'yes'
```

- Check the value of the variable

Close Vim and open it again. It should define this new global variable. We can
check it by entering command mode and running:

```
:echo g:loaded_custom
```

If we did everything correctly, we should see the value `yes` that we defined
in `custom.vim`. After that, we can create as many configuration files as we
want. We will define a logical order for each one and separate them by purpose
so they are easier to edit later.

## Interface

We will start by making the editor friendlier. For that, we will create the
`80-ui.vim` file.

> [!NOTE]
>
> Why `80-`? It is very common to see configuration files arranged this way
> because it lets us control the order in which they run. For example, if we
> want to load a plugin to change our color theme, we need to make sure the
> plugin loads first and only then define the option that selects that theme.
> It would look like this:
>
> ```
> # The 70-* file has priority over 80-*. We can use this logic
> # for any file from 00-* to 99-*.
> conf.d/
> ├── 70-plugins.vim
> └── 80-ui.vim
> ```
>
> We use `80-` because, although it is arbitrary, interface settings are not
> crucial to the program's operation. So, if they take too long to load on a
> low-resource machine, we do not want them to interrupt more important
> settings, such as keyboard shortcuts that we might define in `10-`.

With that in mind, we add the following options to improve the user interface
and make it more useful for common cases:

```vim {filename="~/.config/myvim/conf.d/80-ui.vim"}
set number
set relativenumber
set cursorline
set ignorecase
set smartcase
set wildmode=list:longest
set laststatus=2
set list
set listchars=tab:▏\ ,trail:~
set path+=** " Make Vim's native search recursive
set hidden " Allow switching buffers without saving the current one

" If clipboard support is available,
" map it to the system clipboard so
" we can copy and paste to and from Vim.
if has('clipboard')
  set clipboard=unnamedplus
endif
```

> [!TIP]
> To see a description of what an option does, we can write the following in
> Vim command mode:
>
> ```
> :help <subject>
> ...or, abbreviated:
> :h <subject>
> ```
>
> For example, `:h relativenumber` tells us:
>
> ```
> 		*'relativenumber'* *'rnu'* *'norelativenumber'* *'nornu'*
> 'relativenumber' 'rnu'	boolean	(default off)
> 			local to window
> 	Show the line number relative to the line with the cursor in front of
> 	each line. Relative line numbers help you use the |count| you can
> ...
> ```
>
> To close the help buffer, we can run `:bd`.

## Resume Sessions

It is a bit annoying, especially with very large files, to search again for
the exact place where we were working before closing a file, or to rebuild
Vim's folds after leaving the file and opening it again.

```vim {filename="~/.config/myvim/conf.d/70-restore.vim"}
" Enable undofile to preserve edit history and go back
" to previous changes with `u`, even after closing the file.
set undofile

" Put the cursor in the same position it was in before closing Vim.
augroup RestoreCursor
  autocmd!
  autocmd BufReadPost *
        \ if line("'\"") >= 1 && line("'\"") <= line('$') |
        \   execute 'normal! g`"' |
        \ endif
augroup END

" Preserve folds
augroup SaveView
  autocmd!
  autocmd BufWinLeave *
        \ if &buftype == '' && expand('%') != '' | mkview | endif
  autocmd BufWinEnter *
        \ if &buftype == '' && expand('%') != '' | silent! loadview | endif
augroup END
```

## Shortcuts

Shortcuts are very personal, but here are a couple of common mappings. Note
that we keep them in `10-keymaps.vim` because, as mentioned earlier, we want
to load these mappings first so we can start using the editor even if other
parts of the configuration have not loaded yet:

```vim {filename="~/.config/myvim/conf.d/10-keymaps.vim"}
" In any mode, save the file with Ctrl+S
nnoremap <C-s> :w<CR>
inoremap <C-s> <Esc>:w<CR>a
vnoremap <C-s> <Esc>:w<CR>gv

" Press Shift+Y to copy a line
" from the cursor position onward
nnoremap Y y$
```
