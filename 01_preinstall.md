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
## 📁 Mount the Filesystems

```bash
mount /dev/dropped/root /mnt
mkdir -p /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot
mkdir -p /mnt/home
mount /dev/dropped/home /mnt/home
swapon /dev/dropped/swap
```

or with single logical volume with btrfs sub volumes 
Here’s a Markdown block you can add to your guide as an alternative to using a logical volume for /home. This version keeps the encrypted LVM for swap and root, but uses a Btrfs subvolume for /home instead of a separate LV.

⸻

🧩 Optional: Use a Btrfs Subvolume for /home Instead of an LVM Volume

💡 Use this if you prefer subvolumes over separate LVM logical volumes. It simplifies snapshotting and space management.

🧱 LVM Layout (No LVM home Volume)

lvcreate -L 66G -n swap dropped
lvcreate -l +100%FREE -n root dropped

🗂️ Format Root with Btrfs and Create Subvolumes

mkfs.btrfs -L root /dev/dropped/root
mount /dev/dropped/root /mnt

# Create subvolumes
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home

# Unmount temporary mount
umount /mnt

📁 Mount the Subvolumes

# Mount root subvolume
mount -o subvol=@ /dev/dropped/root /mnt

# Mount boot
mkdir -p /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot

# Mount home subvolume
mkdir -p /mnt/home
mount -o subvol=@home /dev/dropped/root /mnt/home

# Enable swap
mkswap /dev/dropped/swap
swapon /dev/dropped/swap

📜 Add to /etc/fstab Later

You’ll need to add something like this during configuration:

/dev/dropped/root  /      btrfs  rw,relatime,subvol=@      0 1
/dev/dropped/root  /home  btrfs  rw,relatime,subvol=@home  0 2


⸻

Let me know if you’d like this embedded directly in the flow of your current guide (rather than as an optional section).

