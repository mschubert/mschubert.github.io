Arch Linux setup on HP Spectre X360
===================================

Making Windows ready for dual boot
----------------------------------

Partition manager in Windows: make Win smaller
Disable bit locker (Device Encryption Settings)
Install firmwares, updates [HP Support Assistant]

Setup boot sequence
-------------------

F10 Bios setup: disable secure boot, set boot order to first USB

boot arch

Format hard disk
----------------

```
fdisk /dev/nvme0n1
fdisk> n
# accept defaults; this will make a Linux file system on the empty space
fdisk> w
fdisk -l # to check partition was created correctly
mkfs.ext4 /dev/nvme0n1p5 # format partition 5 using ext4 file system
```

```
mount /dev/nvme0n1p5 /mnt
mkdir /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot
```

Connect to the internet on WPA2 wifi
------------------------------------

```
ip link set wlp0s20f3 up
iw wlp0s20f3 link # "not connected"
iw wlp0s20f3 scan | less
wpa_passphrase <YOUR_ESSID> >> /etc/wpa_supplicant.conf
cat /etc/wpa_supplicant.conf
```

```
# reading passphrase from stdin
network={
	ssid="YOUR_ESSID"
	#psk="unencrypted_password"
	psk=<alphanumeric sequence of encrypted password>
}
```

```
wpa_supplicant -B -D wext -i wlp0s20f3 -c /etc/wpa_supplicant.conf
iw wlp0s20f3 link
```

```
dhclient wlp0s20f3
ping google.com
```

Bootstrap the base system
-------------------------

Optional: edit `/etc/pacman.d/mirrorlist` to move a mirror close to you to the
top.

```
# bootstrap base image to the mounted parition(s) on /mnt
pacstrap /mnt base
# write currently mounted UUIDs (-U) to new fstab
genfstab -U /mnt >> /mnt/etc/fstab
```

Here, I change the option for `/` to `rw,noatime` to not write file access
times on the SSD (lower wear).

chroot into the new system
--------------------------

Uncomment required locales in `/etc/locale.gen`.

```
arch-chroot /mnt
# set up time zone
ln -sf /usr/share/zoneinfo/Europe/Amsterdam /etc/localtime
hwclock --systohc
# generate localisation info
locale-gen
```

(left out: etc/hostname, /hosts

```
mkinitcpio -p linux
passwd
```

Installing a boot loader
------------------------

Choosing systemd-boot because it is already installed anyway

```
pacman -S intel-ucode
bootctl --path=/boot install
```

/boot/loader/entries/arch.conf
title   Arch Linux
linux   /vmlinuz-linux
initrd  /intel-ucode.img
initrd  /initramfs-linux.img
options root=/dev/nvme0n1p5

Graphics drivers
----------------

```
pacman -S xf86-video-intel
```

blacklist nouveau because otherwise computer won't shut down/reboot

Desktop environment
-------------------

Add a normal user
add to wheel group
visudo uncomment %wheel

```
pacman -S fish neovim gnome
systemctl enable gdm
```# also: chromium, gnome-shell-ext-{openweather,drop-down-ter-git,sys-mon-git,impatience}, r nvim-r, scipy seaborn, pacman-contrib

vim /etc/gdm/custom.conf
```
WaylandEanble=false
```

Graphics issues: "e"dit start menu entry for Arch, add

```
systemd.unit=multi-user.target
```

to option line
