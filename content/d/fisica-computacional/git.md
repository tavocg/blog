+++
title = '4. Git'
date = '2026-01-10'
tags = ['linux', 'git']
+++

Empezamos por crear una cuenta en [GitHub](https://github.com/), hay otras
plataformas para hostear repositorios git, pero este es el más popular al día de
hoy.

Ahora, creamos un archivo de configuración base para git. No es de mucha
relevancia el usuario e email que escribamos aquí, funciona más que nada como
referencia para saber quién hizo un cambio. Por ejemplo, aquí podría poner
`name = compu1` para identificar que un cambio fue hecho desde la computadora 1.
El archivo debería verse así:

```sh
mkdir -p ~/.config/git
vim ~/.config/git/config
```

```toml {header="~/.config/git/config"}
[user]
	email = miusuario@correo.com # no es necesariamente un correo válido
	name = usuario
```

Ahora, necesitamos una forma de autenticarnos en un servidor git.
Primero, necesitamos crear una llave SSH:

```sh
ssh-keygen -t ed25519
# Podemos simplemente presionar ENTER dos veces hasta que se cree la llave,
# va a pedir nombre de la llave, y una contraseña, esto NO es necesario.
# Aunque se puede definir una contraseña para mayor seguridad,
# ya la seguridad brindada por ssh es más que suficiente.
```

Ahora, necesitamos agregar esta llave en la plataforma de preferencia.
Primero, la imprimimos en la terminal y la copiamos (usualamente con
`Ctrl+Shift+V`)

```sh
cat ~/.ssh/id_ed25519.pub
```

Para agregarla en [GitHub](https://github.com/settings/keys), pegamos la llave
en la pestaña _Ajustes > Llaves SSH y GPG_.

Finalmente, para probar la autenticación exitosa:

```sh
ssh -T git@github.com
# Hi user! You've successfully authenticated, but GitHub does not provide shell access.
```

> [!NOTE]
>
> En caso de obtener un error como:
>
> ```
> Permission denied (publickey).
> ```
>
> Borrar la llave generada y empezar de nuevo esta guía
>
> ```sh
> # CUIDADO!
> # En caso de tener configuraciones previas, respaldar esta carpeta
> rm -r ~/.ssh
> ```
