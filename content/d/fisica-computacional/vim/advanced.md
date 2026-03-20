+++
title = 'Configuración avanzada'
date = '2026-03-20'
tags = ['vim', 'linux']
+++

## Múltiples archivos de configuración

Tener toda nuestra configuración en un solo archivo puede ser tedioso, pero
podemos aprovechar que el lenguaje de configuración de Vim es un lenguaje de
programación completo llamado Vimscript.

Nuestro objetivo será tener una carpeta en el mismo directorio que nuestro
`vimrc`, llamada `conf.d/`. En esta carpeta podremos añadir archivos de
configuración independientes que se carguen cada vez que ejecutamos Vim.
Empezaremos creando una carpeta aislada para no alterar la configuración
existente.

```sh
mkdir -p ~/.config/myvim
touch ~/.config/myvim/vimrc
```

Para utilizar esta configuración, podemos llamar a Vim de la siguiente forma:

```sh
export VIMINIT="source ~/.config/myvim/vimrc"
vim
```

> [!NOTE]
> Puede definir esta variable de entorno en `~/.bashrc` para persistirla entre
> sesiones y utilizar esta configuración por defecto cada vez que inicia Vim.

Ahora que tenemos una manera de acceder a una configuración base aislada y
vacía, empezaremos escribiendo lo siguiente:

```vim {filename="~/.config/myvim/vimrc"}
for file in split(glob(expand('<sfile>:p:h') . '/conf.d/*.vim'), '\n')
  execute 'source' fnameescape(file)
endfor
```

Esta directiva indica que, por cada archivo `conf.d/*.vim` (donde `conf.d` es
una carpeta relativa a nuestro `vimrc` y `*.vim` es un comodín que selecciona
cualquier archivo con el sufijo `.vim`), Vim lo incluirá en la ejecución
actual. Entonces, creamos la carpeta de configuraciones para guardar cada
ajuste en archivos por separado.

Hagamos la prueba:

- Creamos `~/.config/myvim/conf.d/custom.vim`

```sh
mkdir -p ~/.config/myvim/conf.d
touch ~/.config/myvim/conf.d/custom.vim
```

- Definimos una variable global

```sh
vim ~/.config/myvim/conf.d/custom.vim
```

```vim {filename="~/.config/myvim/conf.d/custom.vim"}
let g:loaded_custom = 'yes'
```

- Revisamos el valor de la variable

Cerramos Vim y lo abrimos nuevamente. Debería definir esta nueva variable
global. Podemos revisarlo entrando en el modo comando y ejecutando:

```
:echo g:loaded_custom
```

Si hicimos todo correctamente, deberíamos ver el valor `yes` que definimos en
`custom.vim`. A continuación, podemos crear tantos archivos de configuración
como queramos. Vamos a definir un orden lógico para cada uno y a separarlos
semánticamente para que sea más sencillo editarlos luego.

## Interfaz

Empezaremos haciendo el editor más amigable. Para esto crearemos el archivo
`80-ui.vim`.

> [!NOTE]
>
> ¿Por qué el `80-`? Es muy común ver archivos de configuración con esta
> disposición porque así podemos controlar el orden en el que se ejecutan. Por
> ejemplo, si queremos cargar un plugin para cambiar nuestro tema de colores,
> tenemos que asegurarnos de cargar primero el plugin y luego definir el
> parámetro para seleccionar ese tema. Se vería de esta forma:
>
> ```
> # El archivo 70-* tiene prioridad sobre 80-*. Podemos utilizar esta lógica
> # para cualquier archivo desde 00-* hasta 99-*.
> conf.d/
> ├── 70-plugins.vim
> └── 80-ui.vim
> ```
>
> Se utiliza `80-` porque, aunque es arbitrario, las configuraciones de
> interfaz no son cruciales para el funcionamiento del programa. Entonces, si
> en una computadora de bajos recursos tarda mucho en cargar, no queremos que
> interrumpa otras configuraciones más importantes, como atajos de teclado que
> podríamos definir en `10-`, por ejemplo.

Sabiendo esto, agregamos las siguientes opciones para mejorar la interfaz de
usuario y hacerla un poco más útil para los casos más comunes:

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
set path+=** " Hacer la búsqueda nativa de Vim recursiva
set hidden " Permitir cambiar de buffer sin guardar el actual

" Si tenemos la funcionalidad de clipboard,
" la mapeamos al clipboard del sistema para
" poder copiar y pegar desde y hacia Vim.
if has('clipboard')
  set clipboard=unnamedplus
endif
```

> [!TIP]
> Para ver una descripción de lo que hace una opción, podemos escribir en el
> modo comando de Vim:
>
> ```
> :help <subject>
> ...o también abreviado:
> :h <subject>
> ```
>
> Por ejemplo, `:h relativenumber` nos dice:
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
> Para cerrar el buffer de ayuda podemos ejecutar `:bd`.

## Reanudar sesiones

Es un poco molesto, sobre todo con archivos muy grandes, tener que volver a
buscar la ubicación exacta donde estábamos trabajando antes de cerrar un
archivo, o tener que reconstruir los "folds" de Vim al salir del archivo y
volver a abrirlo.

```vim {filename="~/.config/myvim/conf.d/70-restore.vim"}
" Habilitamos undofile para preservar el historial de modificaciones y volver
" a cambios anteriores con `u`, incluso después de cerrar el archivo.
set undofile

" Ponemos el cursor en la misma posición en la que estaba antes de cerrar Vim.
augroup RestoreCursor
  autocmd!
  autocmd BufReadPost *
        \ if line("'\"") >= 1 && line("'\"") <= line('$') |
        \   execute 'normal! g`"' |
        \ endif
augroup END

" Preservar los folds
augroup SaveView
  autocmd!
  autocmd BufWinLeave *
        \ if &buftype == '' && expand('%') != '' | mkview | endif
  autocmd BufWinEnter *
        \ if &buftype == '' && expand('%') != '' | silent! loadview | endif
augroup END
```

## Atajos

Los atajos son muy personales, pero aquí hay un par de mapeos comunes. Nótese
que los guardamos en `10-keymaps.vim` ya que, como se mencionó antes, nos
gustaría cargar estos mapeos primero para empezar a utilizar el editor aunque
otras partes de la configuración todavía no hayan cargado:

```vim {filename="~/.config/myvim/conf.d/10-keymaps.vim"}
" En cualquier modo, guardar el archivo con Ctrl+S
nnoremap <C-s> :w<CR>
inoremap <C-s> <Esc>:w<CR>a
vnoremap <C-s> <Esc>:w<CR>gv

" Presionar Shift+Y para copiar una línea
" desde la posición del cursor en adelante
nnoremap Y y$
```
