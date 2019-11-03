# https://wiki.archlinux.org/index.php/Installation_guide
# set swiss german keymap
loadkeys sg

# verify efi boot mode
ls /sys/firmware/efi/efivars # this folder should exists

# internet (https://wiki.archlinux.org/index.php/Netctl)
ip link set wlp4s0 down # just to be sure
wifi-menu 

# time
timedatectl set-ntp true

#Partition the disks
# https://wiki.archlinux.org/index.php/GPT_fdisk
# /efi 1G
# / 80G
# /home 125G
# [SWAP] 20G 
# /data Rest

# format them
mkfs.fat -F 32 /dev/sda1
mkfs.ext4 /dev/sda2
mkfs.ext4 /dev/sda3
mkfs.ext4 /dev/sda5
mkswap /dev/sda4
swapon /dev/sda4

# mount them
mount /dev/sda2 /mnt
mkdir /mnt/efi /mnt/home /mnt/data
mount /dev/sda1 /mnt/efi
mount /dev/sda3 /mnt/home
mount /dev/sda5 /mnt/data

#install
pacstrap /mnt base linux linux-firmware

# configure the system
genfstab -U /mnt >> /mnt/etc/fstab
arch-chroot /mnt 
pacman -S e2fsprogs ntfs-3g dosfstools nano vi vim man-db man-pages texinfo tldr sudo tree gparted htop wget thefuck git networkmanager iputils base-devel tar gzip zip unzip
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
hwclock --systohc

#locale
echo KEYMAP=sg > /etc/vconsole.conf
# uncomment in /etc/locale.gen:
#en_GB.UTF-8
#de_CH.UTF-8
locale-gen
# edit /etc/locale.conf
echo "LANG=en_GB.UTF-8" > /etc/locale.conf
echo "LC_NUMERIC=de_CH.UTF-8" >> /etc/locale.conf
echo "LC_TIME=de_CH.UTF-8" >> /etc/locale.conf
echo "LC_MONETARY=de_CH.UTF-8" >> /etc/locale.conf
echo "LC_PAPER=de_CH.UTF-8" >> /etc/locale.conf
echo "LC_MEASUREMENT=de_CH.UTF-8" >> /etc/locale.conf
# hostname
echo "luptop" >> /etc/hostname
echo "127.0.0.1 localhost" >> /etc/hosts
echo "::1       localhost" >> /etc/hosts
echo "127.0.1.1 luptop.localdomain luptop" >> /etc/hosts

# Root pwd
passwd

# Boot loader https://wiki.archlinux.org/index.php/GRUB
pacman -S grub efibootmgr
mkdir /mnt/efi
mount /dev/sda1 /mnt/efi
grub-install --target=x86_64-efi --efi-directory=/mnt/efi --bootloader-id=ARCH_GRUB
grub-mkconfig -o /boot/grub/grub.cfg

# https://wiki.archlinux.org/index.php/Microcode
pacman -S intel-ucode
grub-mkconfig -o /boot/grub/grub.cfg

# complete network config https://wiki.archlinux.org/index.php/Network_configuration
systemctl enable NetworkManager.service
systemctl restart NetworkManager.service
nmtui


# sort mirrors https://wiki.archlinux.org/index.php/Mirrors#Sorting_mirrors
pacman -S pacman-contrib, reflector
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup

rankmirrors -n 30 /etc/pacman.d/mirrorlist.backup > /etc/pacman.d/mirrorlist # Takes a long time
# OR
reflector --latest 200 --protocol https --sort rate --save /etc/pacman.d/mirrorlist

# REBOOT

# user
# install zsh
pacman -S zsh zsh-completions
# useradd -m -G additional_groups -s login_shell username
useradd --create-home --groups adm,ftp,games,http,log,rfkill,sys,wheel,storage --shell zsh lu
passwd lu
# /etc/sudoers allow wheel to use sudo

# deny ssh root login
pacman -S openssh
# /etc/ssh/sshd_config -> PermitRootLogin no

# Reboot and login as lu
sudo pacman -Syu

# install yay
pacman -S binutils, base-devel
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si


# Xorg
# enable multilib: /etc/pacman.conf -> [multilib] Include = /etc/pacman.d/mirrorlist
pacman -S xorg, mesa, lib32-mesa, xterm
# TODO config

# Display Manager
pacman -S lightdm lightdm-gtk-greeter lightdm-gtk-greeter-settings
systemctl enable lightdm.service
# TODO config

# i3
pacman -S i3 dmenu termite ttf-font-awesome xss-lock i3lock network-manager-applet pulseaudio pulseaudio-bluetooth pulseaudio-alsa pavucontrol
# TODO config
setxkbmap ch


# Programs:
yay -S brave-bin
pacman -S geany gnumeric feh gimp vlc gvfs thunar p7zip thunar-media-tags-plugin thunar-archive-plugin thunar-volman file-roller xorg-xmodmap
yay -S teiler
pacman -S telegram-desktop 


# TODO later
# complete network config https://wiki.archlinux.org/index.php/Network_configuration
pacman -S iputils 
# grub https://wiki.archlinux.org/index.php/GRUB/Tips_and_tricks#Play_a_tune
# change in /etc/default/grub
grub-mkconfig -o /boot/grub/grub.cfg




