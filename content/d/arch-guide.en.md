---
date: '2026-01-08'
title: 'Arch Linux Install Guide'
tags: ['linux']
type: docs
---

## Booting the Environment and Preparing the System

If you have a keyboard with a layout other than `us`:

```sh
localectl list-keymaps # List available keymaps
loadkeys my-keymap # Load a keymap
```

### Partitioning Disks

```sh
# List disks:
# fdisk -l

fdisk /dev/sdx
```

The minimum configuration for UEFI is two partitions:

- Partition to be mounted at `/boot` of at least `1GB`
- Partition to be mounted at `/` of at least `23GB`

NOTE: In the rest of the guide, it is assumed that the partitions are:

- `sdx1` is the EFI partition that will be mounted at `/boot`

- `sdx2` will be mounted at `/`

> [!NOTE]
> If using encryption:
>
> ```sh
> cryptsetup luksFormat /dev/sdx2
> cryptsetup open /dev/sdx2 cryptlvm
> ```
>
> We'll need the UUIDs of the encrypted partition and cryptlvm later, so it's a good idea to save them:
>
> ```sh
> # The first one is crypto_LUKS and the second one is the decrypted root partition
> lsblk -f | grep 'crypto_LUKS\|cryptlvm' | awk '{print $4}'
> ```

### Format

```sh
mkfs.ext4 /dev/sdx2
mkfs.fat -F 32 /dev/sdx1
```

### Mount

> [!NOTE]
> If using encryption:
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

# Don't forget the text editor, for example vim or nano
pacstrap -K /mnt vim
```

> [!NOTE]
> If using encryption:
>
> ```sh
> pacstrap -K /mnt cryptsetup lvm2
> ```

## Configure the system

```sh
genfstab -U /mnt >> /mnt/etc/fstab
```

From now on, we will chroot inside the system we are installing, with the command:

```sh
arch-chroot /mnt
```

### Locale

```sh
# e.g. ln -sf /usr/share/zoneinfo/America/Costa_Rica /etc/localtime
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
```

Then:

```sh
hwclock --systohc
```

Edit `/etc/locale.gen` and uncomment the required locales, for example:

```sh {filename="/etc/locale.gen"}
...
# es_CO.UTF-8 UTF-8
# es_CR ISO-8859-1
es_CR.UTF-8 UTF-8
# es_CU UTF-8
...
```

Then, generate them with:

```sh
locale-gen
```

Create the `/etc/locale.conf` file and configure the language by adding the line
(or any other locale from `locale.gen`):

```sh {filename="/etc/locale.conf"}
LANG=es_CR.UTF-8
```

> [!NOTE]
> Persist keyboard layout in `/etc/vconsole.conf`
>
> ```sh {filename="/etc/vconsole.conf"}
> localectl list-keymaps # List available keymaps
> KEYMAP=la-latin1
> ```

### Networking

Define the hostname in `/etc/hostname`, for example: `arch-pc` or the desired hostname.

```sh {filename="/etc/hostname"}
arch-pc
```

Configure `/etc/hosts`, a minimal example configuration:

```sh {filename="/etc/hosts"}
127.0.0.1 localhost
127.0.1.1 arch-pc # (name in /etc/hostname)
::1 localhost
```

```sh
systemctl enable NetworkManager
```

### Users

```sh
useradd -G wheel -m my-username # use preferred username
passwd my-username # set user password
```

Run `visudo` and uncomment the line to enable "wheel" members to use "sudo" (or add it manually if necessary):

```
# %wheel ALL=(ALL:ALL) ALL
```

(Optional) Verify that the `sudo` command can be used when using the new user:

```sh
su - my-username
sudo whoami # should print: root
exit # return to the root user
```

(Optional) Lock the `root` user for security:

```sh
passwd root -l
```

### Initramfs

> [!NOTE]
> If you enable encryption, edit the file `/etc/mkinitcpio.conf` and add the following HOOKS:
>
> ```sh {filename="/etc/mkinitcpio.conf"}
> # Ensure they are in this order and before "filesystems" and "fsck"
> HOOKS=(... encrypt lvm2 ...)
> ```

Create an initial ramdisk

```sh
mkinitcpio -P
```

### Bootloader

Assuming GRUB is used:

> [!NOTE]
> If you enable encryption, modify this line in `/etc/default/grub`:
>
> ```sh {filename="/etc/default/grub"}
> ...
> # cryptdevice is crypto_LUKS and root is the decrypted root partition
> GRUB_CMDLINE_LINUX_DEFAULT="... cryptdevice=UUID=00000000-0000-0000-0000-0000-000000000000:cryptlvm root=UUID=00000000-0000-0000-0000-0000-000000000000"
> ...
> ```

Generate GRUB configuration

```sh
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub
grub-mkconfig -o /boot/grub/grub.cfg
```

## Finish

Exit chroot and reboot

```sh
exit
reboot
```

## (Optional) Post-install

Some extra options in case you need audio, Nvidia drivers, or a desktop environment.

### Audio

Assuming a system with `pipewire` (currently the preferred media server)

```sh
pacman -S pipewire pipewire-alsa pipewire-pulse pipewire-jack wireplumber
```

### Nvidia

```sh
pacman -S nvidia nvidia-utils lib32-nvidia-utils
```

As of today (2024-12-27), it is necessary to add these options to use an Nvidia chip with the proprietary driver.

Modify this line in `/etc/default/grub`

```sh {filename="/etc/default/grub"}
...
GRUB_CMDLINE_LINUX_DEFAULT="... nvidia-drm.modeset=1 nvidia_drm.fbdev=0"
...
```
