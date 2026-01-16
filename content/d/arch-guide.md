---
date: '2026-01-08'
title: 'Guía de instalación de Arch Linux'
tags: ['linux']
---

## Bootear el entorno y preparar el sistema

En caso de tener un teclado con una disposición distinta a `us`:

```sh
localectl list-keymaps # Listar keymaps disponibles
loadkeys mi-keymap     # Cargar un keymap
```

### Particionar discos

```sh
# listar los discos:
# fdisk -l

fdisk /dev/sdx
```

La configuración mínima para UEFI son dos particiones:

- Partición que será montada en `/boot` de al menos `1G`
- Partición que será montada en `/` de al menos `23G`

NOTA: En el resto de la guía, se asume que las particiones

- `sdx1` es la partición EFI que será montada en `/boot`
- `sdx2` será montada en `/`

> [!NOTE]
> En caso de usar encripción:
>
> ```sh
> cryptsetup luksFormat /dev/sdx2
> cryptsetup open /dev/sdx2 cryptlvm
> ```
>
> Necesitaremos los UUIDs de la partición encriptada y de cryptlvm para más adelante, es buena idea guardarlos:
>
> ```sh
> # El primero es crypto_LUKS y el segundo es la partición root desencriptada
> lsblk -f | grep 'crypto_LUKS\|cryptlvm' | awk '{print $4}'
> ```

### Formato

```sh
mkfs.ext4 /dev/sdx2
mkfs.fat -F 32 /dev/sdx1
```

### Montar

> [!NOTE]
> En caso de usar encripción:
>
> ```sh
> mount /dev/mapper/cryptlvm /mnt
> ```

```sh
mount /dev/sdx2 /mnt
```

```sh
mkdir -p /mnt/boot
mount /dev/sdx1 /mnt/boot/
```

### Pacstrap

```sh
pacstrap -K /mnt base linux linux-firmware grub sudo networkmanager efibootmgr

# No olvidar el editor de texto, por ejemplo vim o nano
pacstrap -K /mnt vim
```

> [!NOTE]
> En caso de usar encripción:
>
> ```sh
> pacstrap -K /mnt cryptsetup lvm2
> ```

## Configurar el sistema

```sh
genfstab -U /mnt >> /mnt/etc/fstab
```

A partir de ahora, estaremos dentro del sistema que estamos instalando, con el comando:

```sh
arch-chroot /mnt
```

### Locale

```sh
# e.g. ln -sf /usr/share/zoneinfo/America/Costa_Rica /etc/localtime
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
```

Luego:

```sh
hwclock --systohc
```

Editar `/etc/locale.gen` y descomentar las locales requeridas, por ejemplo:

```sh {filename="/etc/locale.gen"}
...
# es_CO.UTF-8 UTF-8
# es_CR ISO-8859-1
es_CR.UTF-8 UTF-8
# es_CU UTF-8
...
```

Luego, generarlas con:

```sh
locale-gen
```

Crear el archivo `/etc/locale.conf` y configurar el lenguaje agregando la línea
(o cualquier otra locale de preferencia):

```sh {filename="/etc/locale.conf"}
LANG=es_CR.UTF-8
```

> [!NOTE]
> En caso de requerir otro keymap, persistir la configuración en `/etc/vconsole.conf`
>
> ```sh {filename="/etc/vconsole.conf"}
> localectl list-keymaps # Listar keymaps disponibles
> KEYMAP=la-latin1
> ```

### Redes

Definir el hostname en `/etc/hostname`, escribiendo por ejemplo: `arch-pc` o el hostname deseado.

```sh {filename="/etc/hostname"}
arch-pc
```

Configurar `/etc/hosts`, una configuración mínima de ejemplo:

```sh {filename="/etc/hosts"}
127.0.0.1       localhost
127.0.1.1       arch-pc # (nombre en /etc/hostname)
::1             localhost
```

```sh
systemctl enable NetworkManager
```

### Usuarios

```sh
useradd -G wheel -m mi-usuario # utilizar el nombre de preferencia
passwd mi-usuario              # definir la contraseña del usuario
```

Ejecutar `visudo` y descomentar la línea para habilitar a miembros de "wheel" usar "sudo", (o en su defecto, agregarla manualmente):

```
# %wheel ALL=(ALL:ALL) ALL
```

(Opcional) Comprobar que se puede utilizar el comando `sudo` cuando se usa el nuevo usuario:

```sh
su - mi-usuario
sudo whoami # debería imprimir: root
exit # volver al usuario root
```

(Opcional) Bloquear usuario `root` por seguridad:

```sh
passwd root -l
```

### Initramfs

> [!NOTE]
> En caso de habilitar encripción, editar el archivo en `/etc/mkinitcpio.conf` y agregar las siguientes HOOKS:
>
> ```sh {filename="/etc/mkinitcpio.conf"}
> # Asegurarse que estén en este orden y antes de "filesystems" y "fsck"
> HOOKS=(... encrypt lvm2 ...)
> ```

Crear un entorno de disco ram inicial

```sh
mkinitcpio -P
```

### Bootloader

Asumiento que se utiliza GRUB:

> [!NOTE]
> En caso de habilitar encripción, modificar esta línea en `/etc/default/grub`:
>
> ```sh {filename="/etc/default/grub"}
> ...
> # cryptdevice es crypto_LUKS y root es la partición root desencriptada
> GRUB_CMDLINE_LINUX_DEFAULT="... cryptdevice=UUID=00000000-0000-0000-0000-000000000000:cryptlvm root=UUID=00000000-0000-0000-0000-000000000000"
> ...
> ```

Generar configuración de GRUB

```sh
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub
grub-mkconfig -o /boot/grub/grub.cfg
```

## Finalizar

Salir de `chroot` y reiniciar

```sh
exit
reboot
```

## (Opcional) Post-install

Algunas opciones extras en caso de requerir audio, drivers de nvidia, o un entorno de escritorio.

### Audio

Suponiendo un sistema con `pipewire` (el servidor multimedia de preferencia actualmente)

```sh
pacman -S pipewire pipewire-alsa pipewire-pulse pipewire-jack wireplumber
```

### Nvidia

```sh
pacman -S nvidia nvidia-utils lib32-nvidia-utils
```

Hasta la fecha actual (2024-12-27), es necesario agregar estas opciones para utilizar un chip de nvidia junto con el driver propietario.

Modificar esta línea en `/etc/default/grub`

```sh {filename="/etc/default/grub"}
...
GRUB_CMDLINE_LINUX_DEFAULT="... nvidia-drm.modeset=1 nvidia_drm.fbdev=0"
...
```

Agregar/Modificar esta línea en `/etc/mkinitcpio.conf`

```sh {filename="/etc/mkinitcpio.conf"}
...
MODULES=(nvidia nvidia_modeset nvidia_uvm nvidia_drm)
...
```

### GUI

```sh
# KDE Plasma
pacman -S plasma-meta    # Instalar todo el entorno
pacman -S plasma-desktop # Instalar la base solamente

# GNOME
pacman -S gnome gnome-extra # Instalar todo el entorno
pacman -S gnome             # Instalar la base solamente

# i3
pacman -S i3-wm

# Qtile
pacman -S qtile
```
