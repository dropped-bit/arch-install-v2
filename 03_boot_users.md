# 🚀 Arch Linux Installation – Phase 3: Bootloader, Users & Final Setup

## 🥾 Bootloader Setup

### Option 1: GRUB

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
blkid -s UUID -o value /dev/nvme0n1p2
EDITOR=nvim visudo
```

Edit `/etc/default/grub`:

```bash
nvim /etc/default/grub
```

```text
GRUB_CMDLINE_LINUX="cryptdevice=UUID=<UUID>:cryptroot root=/dev/dropped/root"
```

Then:

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

### Option 2: systemd-boot

```bash
bootctl install
mkdir -p /boot/loader/entries
nvim /boot/loader/entries/arch.conf
```

```ini
title   Arch Linux
linux   /vmlinuz-linux
initrd  /intel-ucode.img
initrd  /initramfs-linux.img
options cryptdevice=UUID=<UUID>:cryptroot root=/dev/mapper/dropped-root
```

```bash
nvim /boot/loader/loader.conf
```

```ini
default arch
timeout 3
console-mode max
editor no
```

## 🖥️ Hostname and Hosts Configuration

```bash
echo "dropped-arch" > /etc/hostname
nvim /etc/hosts
```

```ini
127.0.0.1   localhost
::1         localhost
127.0.1.1   dropped-arch.localdomain dropped-arch
```

## 👤 Create a User with Zsh and Sudo Access

```bash
useradd -m -G wheel -s /bin/zsh archuser
passwd archuser
passwd
EDITOR=nvim visudo
```

Uncomment:

```ini
%wheel ALL=(ALL:ALL) ALL
```

## 🔌 Enable Services

```bash
systemctl enable NetworkManager
systemctl enable sshd
```

## 🔚 Final Step

```bash
exit
umount -R /mnt
reboot
```

✅ You're done!
