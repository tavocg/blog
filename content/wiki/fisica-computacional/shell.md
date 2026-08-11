+++
title = '2. Shell'
date = '2026-01-10'
tags = ['linux', 'shell', 'bash']
+++

| Comando | Descripción | Ejemplo |
|-:|-|-|
| [`man`](https://man7.org/linux/man-pages/man1/man.1.html) | Ver manuales de referencia del sistema | `man ls` |
| [`pwd`](https://man7.org/linux/man-pages/man1/pwd.1.html) | Imprimir nombre del directorio | `pwd` |
| [`ls`](https://man7.org/linux/man-pages/man1/ls.1.html) | Listar contenidos del directorio | `ls` |
| [`cd`](https://man7.org/linux/man-pages/man1/cd.1p.html) | [`builtin`] Cambiar el directorio actual | `cd Descargas` |
| [`mkdir`](https://man7.org/linux/man-pages/man1/mkdir.1.html) | Crear directorios | `mkdir carpeta` |
| [`touch`](https://man7.org/linux/man-pages/man1/touch.1.html) | Cambiar marcas de tiempo | `touch ejemplo.txt` |
| [`rm`](https://man7.org/linux/man-pages/man1/rm.1.html) | Eliminar archivos o directorios | `rm ejemplo.txt` |
| [`cp`](https://man7.org/linux/man-pages/man1/cp.1.html) | Copiar archivos y directorios | `cp a.txt a-copia.txt` |
| [`mv`](https://man7.org/linux/man-pages/man1/mv.1.html) | Mover (renombrar) archivos | `mv doc.pdf book.pdf` |
| [`echo`](https://www.man7.org/linux/man-pages/man1/echo.1.html) | [`builtin`] Mostrar una línea de texto | `echo "hola $USER!"` |
| [`cat`](https://man7.org/linux/man-pages/man1/cat.1.html) | Concatenar archivos | `cat a.txt b.txt` |
| [`cut`](https://man7.org/linux/man-pages/man1/cut.1.html) | Eliminar secciones de una línea | `cut -d. -f1 /etc/hosts` |
| [`head`](https://man7.org/linux/man-pages/man1/head.1.html) | Imprimir la primera parte de archivos | `head /etc/passwd` |
| [`tail`](https://man7.org/linux/man-pages/man1/tail.1.html) | Imprimir la última parte de archivos | `tail /etc/passwd` |
| [`wc`](https://man7.org/linux/man-pages/man1/wc.1.html) | Imprime líneas, palabras, bytes de un archivo | `wc -l /proc/cpuinfo` |
| [`uniq`](https://man7.org/linux/man-pages/man1/uniq.1.html) | Reportar u omitir líneas repetidas | `uniq archivo.txt` |
| [`grep`](https://man7.org/linux/man-pages/man1/grep.1.html) | Imprime líneas que coincidan con patrones | `grep 'NAME' /etc/os-release` |
| [`sed`](https://man7.org/linux/man-pages/man1/sed.1.html) | Editor de secuencias: filtrar, transformar texto | `sed 's/N/P/g' /etc/os-release` |
| [`awk`](https://man7.org/linux/man-pages/man1/awk.1p.html) | Lenguaje de escaneo y procesado de patrones | `awk '{ print $2 }' /proc/meminfo` |
| [`ps`](https://man7.org/linux/man-pages/man1/ps.1.html) | Reporta instantánea de procesos actuales | `ps aux` |
| [`du`](https://man7.org/linux/man-pages/man1/du.1.html) | Estimar el uso del espacio de archivo | `du -h archivo.txt` |

`builtin`: Función incorporada en el shell (para efectos de estas guías, en GNU Bash). Por ejemplo, `for`, `break`, `return`, etc.
