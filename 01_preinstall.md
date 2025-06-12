# 🧰 Arch Linux Installation – Phase 1: Pre-Installation

## ✅ Prerequisites

- Boot into the Arch ISO (UEFI mode)
- Confirm internet access is needed (via WiFi)
- Identify your target disk (e.g., `/dev/nvme0n1`) using `lsblk`

## 📶 Connect to WiFi with `iwctl`

```bash
iwctl
```

Inside the interactive shell:

```text
[iwctl] device list
[iwctl] station wlan0 connect <SSID>
[iwctl] exit
```

## ⌨️ Set Console Keyboard Layout

```bash
loadkeys de-latin1
```

## 🧼 Zero the Target Disk (Optional but Recommended)

```bash
dd if=/dev/zero of=/dev/nvme0n1 bs=4M status=progress
```

## 🧱 Partitioning with `gdisk`

```bash
gdisk /dev/nvme0n1
```

| Partition | Type     | Size   | Code  | Notes          |
|-----------|----------|--------|-------|----------------|
| 1         | EFI      | 4 GiB  | ef00  | Boot partition |
| 2         | LUKS/LVM | Rest   | 8300  | Encrypted LVM  |

## 🔐 Set Up Encryption with LUKS

```bash
modprobe dm-crypt
modprobe dm-mod

cryptsetup luksFormat -v -s 512 -h sha512 /dev/nvme0n1p2
cryptsetup open /dev/nvme0n1p2 cryptroot
```

## 📦 Set Up LVM (Volume Group: `dropped`)

```bash
pvcreate /dev/mapper/cryptroot
vgcreate dropped /dev/mapper/cryptroot

lvcreate -L 66G -n swap dropped
lvcreate -L 384G -n root dropped
lvcreate -l +100%FREE -n home dropped
```

## 🗂️ Format Partitions

```bash
mkfs.fat -F32 /dev/nvme0n1p1
mkfs.btrfs -L root /dev/dropped/root
mkfs.btrfs -L home /dev/dropped/home
mkswap /dev/dropped/swap
```
