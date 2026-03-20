+++
title = '3. Vim'
date = '2026-01-10'
tags = ['vim', 'linux']
+++

Vi IMproved, un editor de texto para programadores.

Existen otros editores de texto como `nano`, `neovim` o `emacs`. Todos tienen
objetivos diferentes, pero cumplen la misma función base de editar texto.

Vim viene incluido en la mayoría de distribuciones de Linux, lo cual es muy
conveniente porque es un editor de texto que puede parecer sencillo, pero es
muy potente una vez aprendemos a utilizarlo.

En la mayoría de instalaciones de `vim`, la herramienta `vimtutor` lo
acompaña. Para utilizarla, escribimos en la terminal `vimtutor` y seguimos los
pasos para completar las diferentes tareas que se nos presentan.

Además de aprender a utilizar los atajos de este programa, podemos
configurarlo a nuestro gusto. Aquí hay una configuración sencilla que podemos
aplicar escribiendo en el archivo `~/.config/vim/vimrc`:

```vim {filename="~/.config/vim/vimrc"}
" Al momento de buscar (con la tecla '/'), ignorar mayúsculas y minúsculas.
set ignorecase

" Si al momento de buscar una letra es mayúscula, entonces la búsqueda se
" vuelve sensible ante mayúsculas y minúsculas de nuevo.
set smartcase

" Al momento de buscar, resaltar los caracteres encontrados.
set hlsearch

" En caso de fallos, cierres accidentales o ediciones simultáneas,
" preservar las ediciones en un swapfile aparte, de manera que se puedan
" rescatar cambios perdidos.
set swapfile

" Persistir la bitácora de cambios para poder consultarla al abrir
" un archivo previamente editado y poder retroceder a lo largo de las
" ediciones incluso después de haber cerrado el programa.
set undofile

" Permitir la búsqueda recursiva entre las carpetas de un proyecto.
set path+=**

" Habilitar el uso del mouse para navegar y seleccionar.
set mouse=a

" Información extra en la barra inferior. Para ver exactamente su
" funcionamiento, al igual que cualquiera de estas opciones, se puede ejecutar
" dentro de Vim:
" :help showcmd
" y, al igual que en cualquier buffer de Vim, podemos salir con :q
set showcmd

" Permite cambiar entre buffers sin guardar los cambios
set hidden

" Mostrar ciertos caracteres como:
"   - Espacios extras al final de la línea
"   - Tabs indicando indentación
"   ... entre otros
" Así, caracteres innecesarios como los espacios al final de la línea pueden
" ser eliminados. Además, ayuda a visualizar los niveles de indentación.
set list
```

Para ver otras opciones de configuración, podemos consultar `:help vimrc`.
Además, internet es un recurso muy bueno para personalizar a detalle nuestra
configuración. Por ejemplo:

{{< youtube XA2WjJbmmoM >}}
