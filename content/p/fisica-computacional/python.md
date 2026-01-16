+++
title = '5. Python'
date = '2026-01-10'
tags = ['linux', 'python']
+++

El procedimiento de descargar python varía de acuerdo al sistema operativo que
estemos utilizando, para instalarlo en Debian GNU/Linux (y derivados como
Ubuntu):

```sh
sudo apt install python3 python-is-python3 python3-dev python3-pip python3-venv
```

Necesitaremos varias librerías base para poder trabajar con Python, para
obtener las librerías de forma ordenada podemos:

## Preparar un entorno virtual "base"

```sh
# Este comando:
# Crea un entorno virtual y lo guarda en ~/.local/share/venv/base

python -m venv ~/.local/share/venv/base
```

## Llamar al entorno de manera sencilla

```sh
# OPCIÓN 1
# Agrega una instrucción al bashrc que llama al script activate para activar el
# entorno virtual (como está en el bashrc, correrá automáticamente en nuevas
# instancias de Bash)

echo ". ~/.local/share/venv/base/bin/activate" >> ~/.bashrc

# OPCIÓN 2
# En lugar de siempre activar el mismo entorno, podemos crear un alias "venv"
# que al escribirse activa el entorno "base". Es decir, cada vez que vayamos
# a trabajar en un proyecto de python, escribimos "venv".

echo 'alias venv=". ~/.local/share/venv/base/bin/activate"' >> ~/.bashrc
```

## Instalar librerías

Una vez tengamos activado el entorno, podemos empezar a instalar los paquetes
necesarios para el curso.

```sh
pip install --upgrade pip          # Actualizar pip a la versión más reciente
pip install jupyter                # Podemos instalar un paquete
pip install numpy matplotlib scipy #   ... o varios a la vez
pip install mkdocs mkdocs-material
```
