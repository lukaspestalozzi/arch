This is a more or less step by step guide how I set up my Lenovo Thinkpad Carbon 1.  

# Arch installation
https://wiki.archlinux.org/index.php/Installation_guide
**set swiss german keymap**
```console
loadkeys sg
```
**verify efi boot mode**
```console
ls /sys/firmware/efi/efivars # this folder should exists
```

**internet** (https://wiki.archlinux.org/index.php/Netctl)
```console
ip link set wlp4s0 down # just to be sure
wifi-menu 
```

**time**
```console
timedatectl set-ntp true
```

## Partition the disk
https://wiki.archlinux.org/index.php/GPT_fdisk
/efi 1G
/ 80G
/home 125G
[SWAP] 20G 
/data Rest

**format**
```console
mkfs.fat -F 32 /dev/sda1
mkfs.ext4 /dev/sda2
mkfs.ext4 /dev/sda3
mkfs.ext4 /dev/sda5
mkswap /dev/sda4
swapon /dev/sda4
```

**mount**
```console
mount /dev/sda2 /mnt
mkdir /mnt/efi /mnt/home /mnt/data
mount /dev/sda1 /mnt/efi
mount /dev/sda3 /mnt/home
mount /dev/sda5 /mnt/data
```

## The "install"
```console
pacstrap /mnt base linux linux-firmware
```

## configure the system
```console
genfstab -U /mnt >> /mnt/etc/fstab
arch-chroot /mnt 
pacman -S e2fsprogs ntfs-3g dosfstools nano vi vim man-db man-pages texinfo tldr sudo tree gparted htop wget thefuck git networkmanager iputils base-devel tar gzip zip unzip
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
hwclock --systohc
```

**locale**
```console
echo KEYMAP=sg > /etc/vconsole.conf
```
uncomment in /etc/locale.gen:
en_GB.UTF-8
de_CH.UTF-8
```console
locale-gen
```

**edit /etc/locale.conf**
```console
echo "LANG=en_GB.UTF-8" > /etc/locale.conf
echo "LC_NUMERIC=de_CH.UTF-8" >> /etc/locale.conf
echo "LC_TIME=de_CH.UTF-8" >> /etc/locale.conf
echo "LC_MONETARY=de_CH.UTF-8" >> /etc/locale.conf
echo "LC_PAPER=de_CH.UTF-8" >> /etc/locale.conf
echo "LC_MEASUREMENT=de_CH.UTF-8" >> /etc/locale.conf
```

**hostname**
```console
echo "luptop" >> /etc/hostname
echo "127.0.0.1 localhost" >> /etc/hosts
echo "::1       localhost" >> /etc/hosts
echo "127.0.1.1 luptop.localdomain luptop" >> /etc/hosts
```

**Root pwd**
```console
passwd
```

**Boot loader** https://wiki.archlinux.org/index.php/GRUB
```console
pacman -S grub efibootmgr
mkdir /mnt/efi
mount /dev/sda1 /mnt/efi
grub-install --target=x86_64-efi --efi-directory=/mnt/efi --bootloader-id=ARCH_GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

**Microcode** https://wiki.archlinux.org/index.php/Microcode
```console
pacman -S intel-ucode
grub-mkconfig -o /boot/grub/grub.cfg
```

**complete network config** https://wiki.archlinux.org/index.php/Network_configuration
```console
systemctl enable NetworkManager.service
systemctl restart NetworkManager.service
nmtui
```

**sort mirrors** https://wiki.archlinux.org/index.php/Mirrors#Sorting_mirrors
```console
pacman -S pacman-contrib, reflector
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup
rankmirrors -n 30 /etc/pacman.d/mirrorlist.backup > /etc/pacman.d/mirrorlist # Takes a long time
```
OR
```console
reflector --latest 200 --protocol https --sort rate --save /etc/pacman.d/mirrorlist
```

**REBOOT**
```console
sudo restart
```

## User
**install zsh**
```console
pacman -S zsh zsh-completions
useradd -m -G additional_groups -s login_shell lu
useradd --create-home --groups adm,ftp,games,http,log,rfkill,sys,wheel,storage --shell zsh lu
passwd lu
```
/etc/sudoers -> allow wheel to use sudo

**deny ssh root login**
```console
pacman -S openssh
```
/etc/ssh/sshd_config -> PermitRootLogin no

**Reboot and login as lu then update the system**
```console
sudo restart
sudo pacman -Syu
```

**install yay**
```console
pacman -S binutils, base-devel
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```


**Xorg**
first enable multilib:
/etc/pacman.conf -> [multilib] Include = /etc/pacman.d/mirrorlist
```console
pacman -S xorg, mesa, lib32-mesa, xterm
```


**Display Manager**
```console
pacman -S lightdm lightdm-gtk-greeter lightdm-gtk-greeter-settings
systemctl enable lightdm.service
```

**i3**
```console
pacman -S i3 dmenu termite ttf-font-awesome xss-lock i3lock network-manager-applet pulseaudio pulseaudio-bluetooth pulseaudio-alsa pavucontrol
```
config: https://gitlab.com/lukas.pestalozzi/i3-config


## Programs:
```console
yay -S brave-bin teiler
pacman -S geany gnumeric feh gimp vlc gvfs thunar p7zip thunar-media-tags-plugin thunar-archive-plugin thunar-volman file-roller xorg-xmodmap telegram-desktop alacritty
```
config: https://gitlab.com/lukas.pestalozzi/config-alacritty

**use https://snapcraft.io/**


# TODO later
https://wiki.archlinux.org/index.php/list_of_applications
i3blocks / i3status
logwatch
openconnect
powertop
redshift [nightmode screen]
conky [system monitor]
**editors**
gedit
gnumeric
nano
Master PDF editor
**Others**
dropbox
steam
virtualbox
7zip
etckeeper
InSync
transmission




# Fun stuff one can do
**grub** https://wiki.archlinux.org/index.php/GRUB/Tips_and_tricks#Play_a_tune
change in /etc/default/grub
grub-mkconfig -o /boot/grub/grub.cfg

# General Stuff you search for everytime:
setxkbmap ch

### Caps-Lock -> Ctrl
$ nano /etc/X11/xorg.conf.d/00-keyboard.conf

Section "InputClass"
        Identifier "system-keyboard"
        MatchIsKeyboard "on"
        Option "XkbLayout" "ch"
        Option "XkbOptions" "ctrl:nocaps"
EndSection

OR: setxkbmap -layout ch -option ctrl:nocaps 


### Natural Scrolling
In i3 config:
exec --no-startup-id synclient NaturalScrolling=1 HorizEdgeScroll=1 VertEdgeScroll=1 VertScrollDelta=-111



