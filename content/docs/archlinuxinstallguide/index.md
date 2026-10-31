---
title: 'How To Install Arch Linux'
weight: 1
---

# How to install Arch Linux

This guide shows how to install Arch Linux With KDE Plasma

> ⚠️ **Warning:** Installing Arch using this guide will completely format the target drive and **erase all existing data**. Make sure to back up any important files before proceeding.

---

## What You Will Need

1. A USB drive with Ventoy installed on it (see [create Ventoy drive](../ventoy-usb-creation-guide/))
2. A computer to install Arch onto with at least 20 gigabytes of free space.
3. An active internet connection.

Insert your USB drive into your computer before starting.

---

## How To Get The Arch ISO Image

![Distro Tracker](images/archdistrotracker.png)

1. Go to [https://linuxisochecker.com/top_linux_distros](https://linuxisochecker.com/top_linux_distros)
2. Search for Arch
3. Click on the "Download" link
4. Click on the "Checksum" link
5. Go to [https://linuxisochecker.com](https://linuxisochecker.com)
6. Click "Browse" and choose the Arch ISO image
7. Click "Browse" next to the "Upload Manifest File" and choose the checksum file.
8. Click "Verify ISO Integrity Now"
9. [Create a Ventoy USB Drive](../ventoy-usb-creation-guide)
10. Copy the downloaded Arch ISO image to the Ventoy drive.

## Boot Into Arch Live Image

1. Start your computer and press the relevant function key to bring up the boot menu. ([Boot Menu Keys](../bootkeys/))

2. Choose your USB drive.

3. Choose Arch from the Ventoy menu.

4. Boot into normal mode.

## Install Arch

### Verify The Boot Mode

Check if your system booted into UEFI mode

```
ls /sys/firmware/efi/efivars
```

If it returns a list then it is EFI mode otherwise it is MBR.

This guide assumes EFI. 

### Connect to the internet

#### Wired connections

If you are using a wired ethernet connection then type:

```
ping google.com
```

If it connects and returns results then you have an active connection.

#### Wireless connections

To use WIFI type the following commands:

```
iwdev
```

The above command will tell you the network device such as wlan0.

Type the following commands replacing wlan0 with the network device and <SSID> with the name of the network you want to connect to.

```
iwctl
station wlan0 scan
station wlan0 get-networks
station wlan0 connect <SSID>
exit
```

If the network is secure enter the WIFI password.

### System Clock

Update the system clock using the following command:

```
timedatectl set-ntp true
```

This only sets the time in your live environment. The real time zone changes will be applied when we perform the installation.

### Partitioning

> ⚠️ **Warning:** - Make sure you have backed up before partitioning your drive.

The plan here is to create 2 partitions:

1. EFI partition - used to boot Arch
2. Root partition - contains all the system files and your own personal files

#### Create partitions

Type the following to determine the disks that you have:

```
lsblk
```

You will see a list of drives and partitions:

```
sda -> sda1
    -> sda2
    -> sda3
sdb -> sdb1
    -> sdb2
    -> sdb3
```

Type the following replacing sdX with the drive you want to install Arch onto:

```
fdisk /dev/sdX
```

The first thing we need to do is delete all the current partitions off the drive:

Type p - A list of partitions will be returned
Type d - It will ask you which partition to delete. Let it choose the default and press return.

Repeat typing 'p' to list partitions and 'd' to delete partitions until all the partitions on the drive are gone.

Type g - This will create a new GPT table

Type n - This will create a new partition. 

Press return - This will skip past the start sector
Type +1G and press return - This will create a 1 gigabyte partition for storing your boot loader

Type n - This will create another new partition

Press return - This will skip past the start sector
Press return - This will skip past the end sector and create a new partition using the rest of the disk.

Type w and press return - The changes will be written to disk

#### Format the partitions

Now we have partitions, the next step is to format them.

The first partitions needs to be an EFI partition and EFI partitions are FAT32.

Type the following to format the first partition, replacing sdXN with your drive and partition number: (i.e. /dev/sda1)

```
mkfs.fat -F32 /dev/sdXN
```

For this example we are going to create an EXT4 partition to store Arch.

Type the following to format the second partition, replacing sdXN with your drive and partition number: (i.e. /dev/sda2)

```
mkfs.ext4 /dev/sdXN
```

### Mount the filesystems

To be able to install Arch to the drive you need to mount the partitions to interact with them.

Type the following command to mount the largest partition (second partition): (I.E. /dev/sda2)

```
mount /dev/sdXN
```

For the EFI partition we need to create a new folder and then mount the boot partition to that folder:

```
mkdir -p /mnt/boot
mount /dev/sdXN /mnt/boot
```

### Installing the base ARCH system

The following command is quite long because it is installing a lot of packages required to use Arch:

```
pacstrap /mnt base linux linux-firmware nano htop networkmanager bluez bluez-utils base-devel git konsole sudo
```

So pacstrap is the application used to install the packages and /mnt is the folder where the packages will be installed.

* base - this is the base Arch package
* linux - this is Linux
* linux-firmware - this is a collection of firmware files that Linux device drivers load to make certain hardware work, such as Wi-Fi adapters, Bluetooth devices, and graphics or audio hardware.
* nano - text editor
* htop - this shows a list of processes along with the resource consumption of those processes
* networkmanager - this enables you to connect to the internet
* bluez - bluetooth
* bluez-utils - bluetooth utilities
* base-devel - required for building and installing yay post installation
* git - code management tool
* konsole - a terminal emulator
* sudo - used to provide super user do privileges

### Generate fstab

You need to tell Linux which file systems to mount and where they are and you do this with fstab.

Type the following:

```
genfstab -U /mnt >> /mnt/etc/fstab
```

genfstab is used to detect mounted file system that are under the /mnt folder and then appends the output to /mnt/etc/fstab.

### Switch into your new Arch system (Chroot)

Type the following to switch into your Arch system.

```
arch-chroot /mnt
```

You are now essentially inside your new Arch system even though it can't do much at the moment.

### Configure the new Arch system

#### Timezone and clock

Type the following into the terminal:

```
ln -sf /usr/share/zoneinfo/Europe/London /etc/localtime
```

This creates a symbolic link from /usr/share/zoneinfo/Europe/London to /etc/localtime and in essence sets the timezone to London.

If you don't live in London you will obviously need to use a different command:

To see a list of timezones type the following:

```
ls /usr/share/zoneinfo
```

It will return a list of files and folders.

If for instance you live in Europe then you should type the following command to find all the cities within Europe

```
ls /usr/share/zoneinfo/Europe
```

Now you basically use the first command with the city in which you reside. 

For instance the following will set your timezone to Helsinki in Finland:

```
ln -sf /usr/share/zoneinfo/Europe/Helsinki /etc/localtime
```

Now type the following

```
hwclock --systohc
```

This copies the current system time to the computer's hardware clock (cmos battery).

#### Localisation

Edit the following file:

```
nano /etc/locale.gen
```

Scroll down until you find the keyboard layout for your computer and remove the #.

Type ctrl and o to save and ctrl and x to exit

Type the following to generate the locales based on the file you just edited

```
locale-gen
```

Finally type the following:

```
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

#### Set the hostname

Give your computer a hostname (name the machine):

```
echo "giveyourcomputeraname" > /etc/hostname
```

Edit the hosts file and dd a line

```
nano /etc/hosts
```

```
127.0.0.1 localhost
::1       localhost
127.0.1.1 giveyourcomputeraname.localdomain
```

Press ctrl and o to save and ctrl and x to exit.

Essentially this makes it possible to ping your computer using its name.

#### Set the root password

To set the root password type the following command and enter and repeat the password you want to use for root.

```
passwd
```

#### Install the bootloader (GRUB)

To be able to boot into Arch we need to install a boot loader. There are many different types of bootloader but for this example we will use GRUB.

Type the following:

```
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

The first line installs grub and efibootmgr.

The grub-install command installs grub in the efi folder and sets the bootloader-id to GRUB.

The final line creates the grub config file with all the boot entries.

#### Create a user

To create a user run the following command and replace username with the name you wish to use:

```
useradd -m -G wheel,video,audio,storage -s /bin/bash username
```

This creates a new user and grants permissions to the wheel, video, audio and storage groups and sets the shell to bash. The -m creates all the relevant home folders.

To give the user sudo privileges do the following:

```
EDITOR=nano visudo
```

This will open the sudoers file within nano.

Scroll down until you see the following:

```
#%wheel ALL=(ALL:ALL)
```

Remove the # and save the file by pressing ctrl and o and then exit nano by pressing ctrl and x

#### Installing a desktop and useful software

This is the bit you get to go wild and install the software you need. 

For this example we will use KDE Plasma with some common applications.

Type the following:

```
pacman -S plasma-desktop kscreen plasma-workspace wayland pipewire pipewire-alsa pipewire-pulse wireplumber lightdm lightdm-gtk-greeter ffmpeg x264 x265 gst-plugins-good gst-plugins-bad gst-plugins-ugly gst-libav libva-utils vdpauinfo firefox libreoffice-fresh vlc strawberry kdenlive obs-studio kate dolphin cups cups-pdf gutenprint avahi system-config-printer print-manager plasma-nm bluedevil plasma-pa pavucontrol vlc-plugin-ffmpeg hplip nss-mdns
```

This is a long list of packages but here is what they are:

* plasma-desktop - KDE Plasma desktop environment
* kscreen - KDE display management service
* plasma-workspace - KDE Plasma core tools such as panels etc
* wayland - the graphical desktop
* pipewire - used for audio
* pipewire-alsa - used for ALSA audio
* pipewire-pulse - used for Pulse audio
* wireplumber - used for audio
* lightdm - the desktop manager used for logging in
* lightdm-gtk-greeter - the way lightdm appears
* ffmpeg - video
* x264 - x264 codecs
* x265 - x265 codecs
* gst-plugins-good - these are audio/video plugins that are free and open source
* gst-plugins-bad - these are audio/video plugins that are still in development
* gst-plugins-ugly - these are audio/video plugins that are licensed or have patents (not necessarily open source)
* gst-libav - used to decode and encode other video and audio formats
* libva-utils - a Linux interface VA-API for hardware accelerated video processing
* vdpauinfo - same as libva-utils but for VDPAU
* firefox - web browser
* libreoffice-fresh - productivity suite containing word processor, spreadsheets etc
* vlc - VLC media player
* vlc-plugin-ffmpeg - Encoding to enable you to play all videos using VLC
* strawberry - audio player
* kdenlive- video editing
* obs-studio - video creation
* kate - text editor
* dolphin - file manager
* cups - printing
* cups-pdf - pdf printing
* gutenprint - open source printer drivers
* avahi - provides zero-configuration networking on Linux. It lets devices and apps discover services on the local network without manually configuring addresses, commonly using mDNS and DNS-SD
* system-config-printer - graphical printing utility
* print-manager - printing manager
* plasma-nm - network icon on plasma
* bluedevil - bluetooth icon on plasma
* plasma-pa - audio icon on plasma
* pavucontrol - audio volume control settings
* hplip - print manager for HP printers
* nss-mdns - lets Linux resolve local network names ending in .local using mDNS, such as printer.local or another computer’s local hostname. It integrates with the system’s normal hostname lookup, so apps can use those names without special configuration

#### Enable services

The following will start the network, bluetooth and printing services at startup

```
systemctl enable NetworkManager
systemctl enable bluetooth
systemctl enable cups
systemctl enable avahi-daemon
```

#### Setup the display manager (lightdm)

Type the following to edit the lightdm configuration file:

```
sudo nano /etc/lightdm/lightdm.conf
```

Under the section [LightDM] enter the following

```
start-default-seat=true
wayland-sessions-dir=/usr/share/wayland-sessions
```

Scroll down until you see [Seat:*]

Under this section add the following:

```
greeter-session=lightdm-gtk-greeter
```

Press ctrl and o to save and ctrl and x to exit.

Now start the lightdm service by typing the following:

```
sudo systemctl enable lightdm
```

#### Reboot into your new system

Reboot your computer. 

To do this type the following:

```
exit
umount -R /mnt
reboot
```

When your login screen appears enter your username and password to log in.

#### How to install yay (For the AUR)

> ⚠️ **Warning:** - The AUR should be used sparingly and with extreme caution.

To install yay enter the following commands:

```
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
cd ..
rm -rf yay
```

This clones the yay repository from the Arch repos and builds the package, installs it and then deletes the repository.

#### How to install Microsoft Fonts

To install Microsoft fonts for use with LibreOffice etc type the following:

```
yay -S ttf-ms-fonts
```

#### How to setup Flatpaks

To setup flatpaks in Arch, type the following:

```
sudo pacman -S flatpak
```

Restart your computer.