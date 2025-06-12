# 💾 Arch Linux Installation – Phase 2: Installing the Base System

## 📦 Install Base System

```bash
pacstrap -K /mnt base base-devel linux linux-firmware git neovim lvm2 grub efibootmgr zsh xdg-user-dirs networkmanager intel-ucode openssh
```

## 🧾 Generate `fstab`

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

## 🐚 Enter the New System

```bash
arch-chroot /mnt
```

## 🕒 Set Time Zone and Enable NTP

```bash
ln -sf /usr/share/zoneinfo/Europe/Berlin /etc/localtime
hwclock --systohc
timedatectl set-ntp true
```

## 🗣️ Set Locale

```bash
sed -i 's/^#en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

## ⌨️ Set Console Keymap

```bash
echo "KEYMAP=de-latin1" > /etc/vconsole.conf
```

## 🔐 Configure `mkinitcpio`

```bash
sed -i 's/^HOOKS=.*/HOOKS=(base udev autodetect modconf block encrypt lvm2 filesystems fsck)/' /etc/mkinitcpio.conf
mkinitcpio -P
```
