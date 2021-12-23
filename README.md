# Arch Install Guide

In this guide you will learn how to install arch linux without another
os like windows, the steps for the arch linux installation are:

- Create the arch linux live cd
- Boot the live cd
- Changing the keyboard distribution
- Connect to internet
- Create disk partitions
  - Format and create the partitions types
  - Mount the partitions in the system
- Install the base packages and the base system in the mounted root partition
- Create the fstab file
- Use `arch-chroot` to enter in the arch linux installation from the live cd
- Generate the localtime using the zoneinfo folder
- Generate the clock with the hc
- Generate the locales
- Generate the date config
- Generate the `vconsole.conf` file
- Create the hostname for the system
- Create the hosts file for the system
- Configure sudo for the system
- Create the users
  - Change the password for the root user
  - Create a user with sudo privilegies
    - Add a shell for the user
    - Add the user to the groups
    - Set a password for the user
- Configure networkmanager (enable with `systemctl`)
- Install the grub
  - create the `grub.cfg` file (the grub config file to load arch linux)
- Enter in the new system
- Install gcc and make
- Install xorg
- Install pacman-contrib
- Install yay
- Install lightdm
  - Install lightdm-webkit2-greeter
  - Install lightdm-webkit-theme-aether
    - Activate the lightdm-webkit2-greeter
    - Activate the lightdm-webkit-theme-aether lightdm theme
  - Enable lightdm in the system
- Install a graphic environment (my dotfiles for this example, optional if you don't like it)
  - Clone my [dotfiles](https://github.com/AlphaTechnolog/dotfiles)
  - Install the required programs
  - Install my dotfiles with the installer
    - Create the config symlinks
    - Create the custom scripts symlinks
    - Create the rofi config
    - Create the home symlinks
    - Make dwm
      - Create the dwm xsession .desktop file
  - Change the default shell to fish
  - Configure the `~/.xprofile` file
    - xset led (optional if your keyboard doesn't have leds)
    - xrandr
    - nm-applet
    - volumeicon
    - cbatticon
    - udiskie
    - path
    - env variables
    - feh
  - Configure tmux
    - Install powerline support for tmux
  - Configure Neovim
    - Install packer
    - Install all neovim plugins
    - Install languages servers
      - Install nodejs / npm packages
      - Install python packages
- Install another graphic environment (gnome and xfce)
  - Install gnome (gnome gnome-extra)
  - Install xfce (xfce)
- Reboot and enjoy

## Create the livecd

To create the livecd first download the iso in the archlinux
website, and then connect an usb to format it with the livecd
of archlinux, to do this task you can use programs like rufus for windows
or unetboot for linux

## Boot the livecd in your machine

Connect the booted usb in your machine and change in your bios
the run priority to the usb, restart the machine

## Changing the keyboard distribution

If you want to use another keyboard distribution
during the installation, you can do this command:

```sh
loadkeys dist # like es, en or another
```

## Connect to internet

To connect to internet in arch linux you can do it with the
`iwctl` utility, please refer to it documentation to connect to internet

## Create disk partitions

To make the partitions, you can use `fdisk` or `cfdisk` utilities
that are installed by default in arch linux livecd.

First identify the name of your disk with fdisk:

```sh
fdisk -l
Disk /dev/sda: 238.47 GiB, 256060514304 bytes, 500118192 sectors
Disk model: XSTAR SSD 256GB 
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: CB47A32F-BEC1-F249-945E-1F1EE125BCD1
```

In my case the disk is `/dev/sda` (the more common disk name)

To create the partitions for **UEFI support**, do the next steps:

```sh
cfdisk /dev/sda
```

First select the disk label type, for the uefi/efi support the recommended
is the `gpt` label disk type.

And then create with the menus a partition with type `EFI System`
and with size of `512M` that is the recommended size for the efi boot partition.

The next partition is the root partition, it will be a `Linux root (x86-64)`
partition and in my case it has `234G` of size, it's the size of the `/`
for your personal files, it includes `/home`.

The next partition is for the swap, it will be have of size, the size of
your ram memory divided by 2, in my case I have `8G` of ram, the size for
the partition will be `4G` and the type will be `Linux swap`.

**Write all changes of the partitions with cfdisk**

And if you run `fdisk -l /dev/sda`, you will get a result like this:

```sh
fdisk -l /dev/sda
Disk /dev/sda: 238.47 GiB, 256060514304 bytes, 500118192 sectors
Disk model: XSTAR SSD 256GB 
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: CB47A32F-BEC1-F249-945E-1F1EE125BCD1

Device         Start       End   Sectors  Size Type
/dev/sda1       2048   1050623   1048576  512M EFI System
/dev/sda2    1050624 491784191 490733568  234G Linux root (x86-64)
/dev/sda3  491784192 500118158   8333967    4G Linux swap
```

### Format and create the partitions types

At this moment we have three partitions, the partition
for the boot, the partition for the root
and the partition for the swap

First format and create a vfat partition with the next
command, in my case the partition name is `/dev/sda1`,
`fdisk` show it:

```sh
fdisk -l /dev/sda
Disk /dev/sda: 238.47 GiB, 256060514304 bytes, 500118192 sectors
Disk model: XSTAR SSD 256GB 
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: CB47A32F-BEC1-F249-945E-1F1EE125BCD1

Device         Start       End   Sectors  Size Type
/dev/sda1       2048   1050623   1048576  512M EFI System
/dev/sda2    1050624 491784191 490733568  234G Linux root (x86-64)
/dev/sda3  491784192 500118158   8333967    4G Linux swap
```

Is this line

```
/dev/sda1       2048   1050623   1048576  512M EFI System
```

It have an `EFI System` partition type and `512M`, format it:

```sh
mkfs.vfat /dev/sda1
...
```

Then format with the type `ext4` the root partition:

```
/dev/sda2    1050624 491784191 490733568  234G Linux root (x86-64)
```

```sh
mkfs.ext4 /dev/sda2
```

And then the same with the swap partition but the command is another:

```
/dev/sda3  491784192 500118158   8333967    4G Linux swap
```

Format in and active the swap with the `swapon` command

```sh
mkswap /dev/sda3
swapon /dev/sda3
```

### Mount the partitions in the system

First mount the root partition in `/mnt` of the livecd:

```sh
mount /dev/sda2 /mnt
```

Then create the folder `/mnt/boot` and mount the boot
partition in it folder:

```sh
mount /dev/sda1 /mnt/boot
```

## Install the base packages and the base system in the mounted root partition

To install the base packages and the system in the mounted
partition, you can use a script named `pacstrap`, it install
the base system and the packages that you specify with pacman in the
specified mounted partition, make an installation like this:

```sh
pacstrap /mnt base linux linux-firmware grub networkmanager sudo vim
```

It will create the traditional linux files structure in `/mnt`
like this:

```sh
wxr-xr-x - root root 21 Dec 12:29 dev
drwxr-xr-x - root root 21 Dec 14:46 etc
drwxr-xr-x - root root 18 Dec 14:52 home
lrwxrwxrwx 7 root root  6 Dec 22:41 lib -> usr/lib
lrwxrwxrwx 7 root root  6 Dec 22:41 lib64 -> usr/lib
drwx------ - root root 18 Dec 14:10 lost+found
drwxr-xr-x - root root  6 Dec 22:41 mnt
drwxr-xr-x - root root 21 Dec 14:44 opt
dr-xr-xr-x - root root 21 Dec 12:29 proc
drwxr-x--- - root root 21 Dec 14:28 root
drwxr-xr-x - root root 21 Dec 12:30 run
lrwxrwxrwx 7 root root  6 Dec 22:41 sbin -> usr/bin
drwxr-xr-x - root root 18 Dec 14:48 srv
dr-xr-xr-x - root root 21 Dec 12:29 sys
drwxrwxrwt - root root 21 Dec 16:34 tmp
drwxr-xr-x - root root 19 Dec 15:54 usr
drwxr-xr-x - root root 19 Dec 16:31 var
```

And then install the next packages in the partition with
pacman:

- base: This is the base arch linux packages
- linux: This is the kernel of the system
- linux-firmware: This is the firmware for the system
- grub: This is the grub for the system, it will copy the os into the ram and then execute the system
- networkmanager: This provide internet for you
- vim: vim... if you don't know vim. ok xd, switch to nano xd, it basically make the same of nano.
- sudo: Sudo allow to execute commands as root user for someones user that root granted the permissions

**This process will take a lot of time depending with your network speed**

## Create the fstab file

To generate the fstab file, you can use the `genfstab` utility:

```sh
genfstab -U /mnt >> /mnt/etc/fstab
```

It will generate the fstab file for `/mnt` and then
append it to the `/etc/fstab` file, like this:

```
# Static information about the filesystems.
# See fstab(5) for details.

# <file system> <dir> <type> <options> <dump> <pass>
# /dev/sda2
UUID=2785f9d8-4bfa-463e-92e3-5888012a563b	/         	ext4      	rw,relatime	0 1

# /dev/sda1
UUID=3EF7-850E      	/boot     	vfat      	rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=ascii,shortname=mixed,utf8,errors=remount-ro	0 2

# /dev/sda3
UUID=a51d2394-b5f3-4bd2-8294-c094b33188ef	none      	swap      	defaults  	0 0
```

It's an example fstab file for my system, that `genfstab` generate for me.

## Enter to the installation using arch-chroot

Enter to the installed system with `arch-chroot`:

```sh
arch-chroot /mnt
```

## Generate the localtime using the zoneinfo folder

Into the system, run the next command to create the localtime file:

```sh
ln -sf /usr/share/zoneinfo/REGION/CITY /etc/localtime
```

## Generate the clock with the hc

Now make this command:

```sh
hwclock --systohc
```

It will use the `/etc/localtime` file

## Generate the locale

In my case for english, uncomment this in the file
`/etc/locale.gen` file:

```
en_US.UTF-8 UTF-8
```

And then run the command:

```sh
locale-gen
```

## Generate the date config

To do this, run this command

```sh
hwclock -w
```

## Create the vconsole.conf file

If you run `loadkeys` at the start of the installation
to make this changes permanent, do this modification in the `vconsole.conf` file:

```sh
touch /etc/vconsole.conf
vim /etc/vconsole.conf
```

```sh
KEYMAP=us # or es, depending
```

## Create the hostname file

This define the hostname of your system, this is
simple, write the hostname in the `/etc/hostname` file:

```sh
echo myepichostname > /etc/hostname
```

## Create the hosts file

The `/etc/hosts` file, copy this content:

```sh
vim /etc/hosts
```

```
# Static table lookup for hostnames.
# See hosts(5) for details.

127.0.0.1 localhost
::1 localhost
127.0.0.1 myepichostname.localdomain myepichostname
```

Where `myepichostname` is the hostname in the
`/etc/hostname` file

## Configure sudo for the system

Edit the sudoers file:

```sh
vim /etc/sudoers
```

Uncomment this if you want the users in the `wheel`
group can use the sudo with password

```
## Uncomment to allow members of group wheel to execute any command
%wheel ALL=(ALL) ALL
```

Or uncomment this to allow members of group wheel to execute
sudo commands without password

```
## Same thing without a password
%wheel ALL=(ALL) NOPASSWD: ALL
```

## Create the users

First make this, please, if you don't do this
you can't enter to your computer, first remember,
change the root password, **please**

### Change the password for the root user

```sh
passwd
# write two times the root password, a simple password like "root"
```

### Create a user with sudo privilegies

First create the user with his home folder:

> NOTE: The user for my system is called `gabriel`

```sh
useradd -m gabriel
```

#### Add a shell for the user

```sh
usermod --shell /bin/bash gabriel
```

#### Add the user to the groups

```sh
usermod -aG wheel,audio,video gabriel
```

#### Set a password for user

```sh
passwd gabriel
# Write two times the password for the user
```

## Configure the networkmanager

Enable it with `systemctl`, it's important you have internet
this step is **VERY** important.

```sh
systemctl enable NetworkManager.service
```

> NOTE: If you skip this step and you enter to the system you can do this command in the system

```sh
systemctl enable --now NetworkManager.service
```

## Install the grub

To install the grub, remember, we have a mounted partition for the boot
in the `/boot`, it's important to install the grub.

Do this:

```sh
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```

Where `--efi-directory` value is the directory where you mount the boot partition

### Create the grub config file

To create the grub config file you can execute this command:

```sh
grub-mkconfig -o /boot/grub/grub.cfg
```

## Enter in the new system

To enter in the new system, first exit of the chroot
using the command `exit` and then reboot after umount /mnt:

```sh
root@arch-chroot# exit
root@livecd# umount -R /mnt
root@livecd# reboot
```

Then if all is installed successfully, you can view the grub in action
and then boot `Arch Linux`, write your user and your password.

Done? Ok continue.

## Install gcc and make

It's necesary to make the sources
Install it with pacman:

```sh
sudo pacman -S gcc make
```

## Install pacman-contrib package

The `pacman-contrib` package is necesary for
the correct work of some applications:

```sh
sudo pacman -S pacman-contrib
```

It has programs like `checkupdates` and others

## Install xorg

You can install `xorg` with `pacman`:

```sh
sudo pacman -S xorg
```

**This will take a lot of time depending with your network speed**

## Install yay

Yay is an aur helper to install aur packages more easy.

To install yay you need git, install it:

```sh
sudo pacman -S git
```

And then, clone yay.

> I use a folder named `repo` to all clones of some repositories:

```sh
mkdir -p repo
cd ~/repo
git clone https://aur.archlinux.org/yay.git yay
cd yay
```

Then recompile it with `makepkg`:

```sh
makepkg -si
```

**This will take a lot of time depending with your network connection**

## Install lightdm

In this guide we use lightdm as display manager:

```sh
sudo pacman -S lightdm
```

Right, lightdm is only a framework without interface
that works with greeters, install the webkit2 greeter.

### Install lightdm-webkit2-greeter

This is the greeter for this guide, install it with `pacman`:

```sh
sudo pacman -S lightdm-webkit2-greeter
```

Then the webkit2 greeter works with a theme, install the aether theme, it's very beautiful,
you can install another, but I love it.

### Install lightdm-webkit-theme-aether

To install this, we need yay or another aur helper.

```sh
yay -S lightdm-webkit-theme-aether
```

#### Activate the lightdm-webkit2-greeter

This will be activated automatically, but check it
in the file `/etc/lightdm/lightdm.conf`:

```sh
vim /etc/lightdm.conf
```

Check if this line is present:

```
[Seat:*]
...
greeter-session = lightdm-webkit2-greeter
```

If not add this:

```
greeter-session = lightdm-webkit2-greeter
```

To the `[Seat:*]` section

#### Activate the lightdm-webkit-theme-aether

The theme will be activated automatically, but check it
in the file `/etc/lightdm/lightdm-webkit2-greeter.conf`

```sh
vim /etc/lightdm/lightdm-webkit2-greeter.conf
```

Check if this line is present:

```sh
[greeter]
...
theme = lightdm-webkit-theme-aether
```

If not add this to the `[greeter]` section:

```
theme = lightdm-webkit-theme-aether
```

### Enable lightdm in the system

Enable lightdm with `systemctl`:

```sh
systemctl enable lightdm.service
```

## Install a graphic environment (my dotfiles)

You can skip this step if you don't like my
dotfiles and install another desktop manager like gnome
or xfce (the next step)

### Clone the dotfiles in /home/user/.dotfiles

The path `/home/user/.dotfiles` is the default for the
dotfiles installer:

> In my case user is gabriel

```sh
git clone https://github.com/AlphaTechnolog/dotfiles ~/.dotfiles
```

### Install the required programs

This guide only installs qtile and dwm, but this dotfiles
have configs for xmonad, xmobar, bspwm, dwm, spectrwm,
polybar and another programs:

Install the programs:

- qtile
- kitty
- fish
- starship
- nerd-fonts-ubuntu-mono
- neovim
- neovim-symlinks
- firefox
- network-manager-applet
- cbatticon
- udiskie
- volumeicon
- feh
- xorg-xinit
- node-lts-gallium
- npm
- python-pip

Remove:

- vim
- vim-symlinks

```sh
sudo pacman -R vim vim-symlinks
sudo pacman -S qtile feh xorg-xinit \
  node-lts-gallium npm volumeicon \
  udiskie \
  cbatticon \
  network-manager-applet \
  firefox \
  neovim-symlinks \
  neovim \
  fish \
  kitty \
  qtile \
  python-pip
yay -S nerd-fonts-ubuntu-mono
```

### Install the dotfiles with the installer

Run the installer in the dotfiles folder:

```sh
cd ~/.dotfiles
./installer.sh
```

#### Create the config symlinks

Select and execute the create .config symlinks

#### Create the custom scripts symlinks

Select and execute the create .local/bin symlinks

The install the next dependencies:

- brightnessctl
- pavucontrol
- pulseaudio

```sh
sudo pacman -S brightnessctl pavucontrol pulseaudio
```

#### Create the home symlinks

Select and execute the create home symlinks

#### Make dwm

Select the make dwm step

##### Create the xsession .desktop file

Copy and paste this content in `/etc/xsessions/dwm.desktop`:

```sh
sudo mkdir -p /etc/xsessions
sudo vim /etc/xsessions/dwm.desktop
```

```
[Desktop Entry]
Name=Dwm
Comment=Spawn a dwm session
Exec=dwm
Type=Application
Keywords=wm;tiling
```

### Change the default shell to fish

Change the default shell to fish for the user, in my
case "gabriel":

```sh
sudo usermod --shell /usr/bin/fish gabriel
```

Another method to change it is with `chsh`:

```sh
chsh -s /usr/bin/fish
```

### Configure the .xprofile file

This file is executed at login by `xorg-xinit`:

```sh
touch ~/.xprofile
echo '#!/bin/sh' > ~/.xprofile
chmod +x ~/.xprofile
vim ~/.xprofile
```

#### Configure xset

Put this line to `.xprofile`:

```sh
xset led
```

#### Configure xrandr

This is variable depending with your monitor config, in my
case:

```sh
xrandr --output LVDS-1 --mode 1366x768 --primary
```

It works for me, but refers to the `man` documentation of xrandr:

```sh
man xrandr
```

#### Configure system tray utilities

Put this in `.xprofile`:

```sh
nm-applet &
cbatticon -u 5 &
udiskie -t &
volumeicon &
```

#### Configure the path

Put this in the `.xprofile`:

```sh
export PATH="$HOME/.local/bin:$PATH"

YARN_PATH="$HOME/.yarn/bin"

if test -d $YARN_PATH; then
    export PATH="$PATH:$YARN_PATH"
fi
```

#### Configure environment variables

```sh
export TERM="xterm-256color"
```

#### Configure feh

This is the method for the tiling window managers to set
a wallpaper, but you can use too [wl](https://github.com/AlphaTechnolog/wl)
that is a wallpaper manager, that I create

Put this into the `.xprofile` to set your wallpaper at
init the system

```sh
feh --bg-scale picture-url
```

### Configure tmux

First install tmux:

```sh
sudo pacman -S tmux
```

#### Install powerline support for tmux

Install the powerline package, and the dotfiles, setup
all other:

```sh
sudo pacman -S powerline
```

> Then spawn tmux

### Configure neovim

#### Install packer

First install packer.nvim, please first refers to it documentation

#### Install neovim plugins

```sh
vim # this spawn neovim with the neovim-symlinks package
```

And then execute the command `:PackerInstall` inside neovim

#### Install languages servers

##### Install nodejs / npm packages

```sh
sudo npm install -g yarn
sudo npm install -g diagnosticls typescript typescript-language-server vls
```

##### Install python packages

```sh
python -m pip install 'python-language-servers[all]'
```

## Install another graphic environment

If you don't like my dotfiles, you can install
xfce or gnome, or another like lxde, kde plasma,
but in this guide, we only install gnome and xfce.

### Install gnome

To install gnome install the next packages with pacman:

```sh
sudo pacman -S gnome gnome-extra
```

### Install xfce

To install xfce install the next packages with pacman:

```sh
sudo pacman -S xfce
```

And then in lightdm, gnome and xfce will be appear
in the sessions of lightdm.

## Reboot and enjoy

This is all for this guide, now reboot the system and try
all, if you have any issue you can open an issue in this
repository or see more documentation in the arch linux wiki:

```sh
sudo reboot
```

Then login and **enjoy** with your new operative system.
