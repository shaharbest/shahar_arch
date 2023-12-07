# Shahar's Arch Evironment

## WIFI

to see interfaces

```sh
ip addr show
```

```sh
iwctl
```

in the iwctl session

```sh
device list
station <device> scan
station <device> get-networks
station <device> connect <wireless-network-name>
```

## Partitions

inspect current layout

```sh
fdisk -l
```

```sh
fdisk /dev/nvme0n1
```

fdist session

```
p (to see the current layout)
g (to set gpt)
n (new partition)
1 (default)
+500M (size of the patition for UEFI)
t (set partition type)
1 (to choose type UEFI)
n (new partition)
2 (default)
[Enter] (default)
t (set partition type)
2 (default)
30 (for LVM type)
p (to see the current layout)
w (to write the layout)
```

## LVM

```sh
mkfs.fat -F32 /dev/nvme0n1p1
pvcreate --dataalignment 1m /dev/nvme0n1p2
vgcreate volgroup0 /dev/nvme0n1p2
lvcreate -l 100%FREE volgroup0 -n lv_root
modprobe dm_mod
vgscan
vgchange -ay
```

```sh
mkfs.ext4 /dev/volgroup0/lv_root
mount /dev/volgroup0/lv_root /mnt
mkdir /mnt/etc
genfstab -U -p /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab
```

## CHROOT

```sh
pacstrap -i /mnt base
arch-chroot /mnt
pacman -S linux-lts linux-lts-headers linux-firmware
pacman -S neovim
pacman -S base-devel openssh
systemctl enable sshd
pacman -S networkmanager wpa_supplicant wireless_tools netctl
pacman -S dialog
systemctl enable NetworkManager
pacman -S lvm2
nvim /etc/mkinitcpio.conf
# Add "lvm2" in between "block" and "filesystems"
# HOOKS=(base udev autodetect modconf block lvm2 filesystems keyboard fsck)
mkinitcpio -p linux-lts
```

```sh
nvim /etc/locale.gen # (uncomment en_US.UTF-8)
locale-gen
```

Passwords and user

```sh
passwd
useradd -m -g users -G wheel shahar
passwd shahar
pacman -S sudo
```

```sh
EDITOR=nvim visudo
```

uncomment the following line

```txt
%wheel ALL=(ALL) ALL
```

### GRUB

```sh
pacman -S grub efibootmgr dosfstools os-prober mtools
mkdir /boot/EFI
mount /dev/nvme0n1p1 /boot/EFI
grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck
mkdir /boot/grub/locale
cp /usr/share/locale/en\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo
grub-mkconfig -o /boot/grub/grub.cfg
cat /etc/fstab # TEST
```

exit chroot

```sh
exit
umount -a
reboot
```

## POST install

swap file

```sh
dd if=/dev/zero of=/swapfile bs=1M count=2048 status=progress
chmod 600 /swapfile
mkswap /swapfile
cp /etc/fstab /etc/fstab.bak
echo '/swapfile none swap sw 0 0' | tee -a /etc/fstab
```

time zone

```sh
timedatectl set-timezone America/New_York
systemctl enable systemd-timesyncd
```

```sh
hostnamectl set-hostname archpad
```

```sh
nvim /etc/hosts
```

add this content:

```txt
127.0.0.1 localhost
127.0.1.1 archpad
```

```sh
pacman -S intel-ucode mesa
```

## [LARBS](https://larbs.xyz/)

```sh
curl -LO larbs.xyz/larbs.sh
sh larbs.sh
```
