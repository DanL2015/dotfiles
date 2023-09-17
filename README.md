# dotfiles
Personal Arch Linux dotfiles and installation instructions so I can restore my system after I clean reinstall
# Arch Installation
(Commands in `**` should be substituted with their correct values)
## PreInstallation
### Wifi
Use `iwctl` to connect to an available wifi network.
```
iwctl
device list
station device scan
station *device* list-networks
station *device* connect *SSID*
```
Verify connection with `ping`.
```
ping 8.8.8.8
```
### System Clock
Enable network time synchronization.
```
timedatectl set-ntp true
```
### File Systems
Use lsblk to determine the partition names.
```
lsblk
```
Use `cfdisk` to create three partitions.
- Boot partition (EFI) of 512 MiB
- Swap partition (swap) of 16 GiB (size of memory for hibernation)
- Root partition (ext4) of remaining size
```
cfdisk *disk*
```
Format partitions.
```
mkfs.ext4 */dev/root_partition*
mkswap */dev/swap_partition*
mkfs.fat -F 32 */dev/efi_system_partition*
```
Mounting the file systems.
```
mount */dev/root_partition* /mnt
mount --mkdir */dev/efi_system_partition* /mnt/boot
swapon */dev/swap_partition*
```
## Installation
### Base packages
Use `pacstrap` to install the base package group.
```
pacstrap -K /mnt base base-devel linux linux-firmware linux-headers vim intel-ucode reflector mtools dosfstools
```
## Configure the System
### Fstab
Generate Fstab file to define the partition mounting.
```
genfstab -U /mnt >> /mnt/etc/fstab
```
### Chroot
Change root into new system
```
arch-chroot /mnt
```
### Time Zone
Set up your time zone.
```
ln -sf /usr/share/zoneinfo/*Region/City* /etc/localtime
hwclock --systohc
```
### Locales
Set up your locales.
Uncomment `en_US.UTF-8 UTF-8` and other locales if needed.
```
vim /etc/locale.gen
```
Generate the locales and add `en_US.UTF-8` to `locale.conf`
```
locale-gen
echo LANG=en_US.UTF-8 > /etc/locale.conf
```
### Set Hostname
```
echo *hostname* > /etc/hostname
```
### Set Hosts
```
vim /etc/hosts
```
Add the following to the file
```
127.0.0.1    localhost
::1          localhost
127.0.1.1    *hostname*
```
### NetworkManager
Install and enable NetworkManager so your wifi works after rebooting.
```
pacman -S networkmanager
systemctl enable NetworkManager
```
### Root password
```
passwd
```
### Install GRUB
```
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```
Optional: detect other operating systems
```
pacman -S os-prober
grub-mkconfig -o /boot/grub/grub.cfg
```
### Reboot
```
exit 
umount -a
reboot
```
## Post Reboot
Login as root using the password you set using `passwd`
### Add new user
```
useradd -mG wheel *user*
passwd *user*
visudo
```
Uncomment the following line:
```
#%wheel ALL=(ALL) ALL
```
Logout from root and relogin with your new user account.
```
exit
```

# Install Software
## Installing AwesomeWM
### Install Xorg and GPU drivers
```
sudo pacman -Syu xorg nvidia nvidia-settings
```
### Install yay for AUR helper
```
sudo pacman -Syu git
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```
### Install AwesomeWM and xinit
```
sudo pacman -Syu xorg-xinit
yay -Syu awesome-git
```
Configure `~/.xinitrc`
```
sudo cp /etc/X11/xinit/xinitrc ~/.xinitrc
sudo vim ~/.xinitrc
```
Remove the last few lines and replace with
```
exec awesome
```
## Installing Useful Applications
### General useful packages
```
sudo pacman -Syu xorg-xclipboard xorg-xinput xorg-xrandr xorg-xrdb acpilight
```
### Useful applications
```
sudo pacman -Syu firefox thunar thunar-archive-plugin thunar-volman xarchiver kitty bleachbit maim timeshift viewnoir gimp gparted gpick zathura zathura-pdf-mupdf code neovim libreoffice-fresh obsidian mpv obs unzip
yay -Syu spotify slack-desktop
```
### Fonts
```
yay -Syu ttf-iosevka-nerd ttf-material-design-iconic-font ttf-ms-win11-auto ttf-roboto noto-fonts-emoji
```
### Audio and Bluetooth
```
sudo pacman -Syu alsa-utils bluez bluez-utils blueman pipewire wireplumber pipewire-audio pipewire-pulse pipewire-alsa
sudo systemctl enable bluetooth.service
systemctl enable --user pipewire-pulse.service
```
### Rice
```
sudo pacman -Syu htop neofetch fish picom ranger playerctl starship
yay -Syu i3lock-color spicetify-cli pywal-16-colors
sudo chsh -s /usr/bin/fish
```
### Optimization
```
sudo pacman -Syu powertop cpupower auto-cpufreq
```


