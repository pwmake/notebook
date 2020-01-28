---
layout: post
title: How To Install Arch Linux
date: 2018-02-07
tags: [HowTo]
feature-img: "https://images.pexels.com/photos/34658/pexels-photo.jpg?cs=srgb&dl=blogging-computer-desk-34658.jpg&fm=jpg"
---


Install Arch Linux step by step, making you understand Linux system more effectively.

<!--more-->

## USB Disk

Recommend to use `dd` command create bootable usb, or other methods click [Here](https://wiki.archlinux.org/index.php/USB_flash_installation_media).

```bash
$ dd bs=4m if=/path/to/archlinux.iso of=/dev/sdx && sync
```

## Install Arch

> Read this [installation guide](https://wiki.archlinux.org/index.php/Installation_guide) first !

Check the boot type is UEFI or MBR.

```bash
# echo nothing means MBR
$ ls /sys/firmware/efi/efivars
```

Make sure network is OK.

```bash
$ ping -c 5 www.so.com
```

Use wifi network,  `recommend to use wire network`

```bash
 # connect to wifi network
$ wifi-menu
```

Enable ssh service to remote control.

```bash
$ systemctl start sshd
```

Set timezone and time rsync.

```bash
$ timedatectl set-ntp true
$ timedatectl set-timezone Asia/Shanghai
```

Set most fast pacman mirrors

```bash
$ pacman-mirror --country 'China'
```

Then  `disk management`

Check disk partition.

```bash
$ lsblk /dev/sdx
```

Use `fdisk` command to manage your paritions.

> Sectors formula：1024 x 1024 x 1024 / 512 x Gigabyte = total sectors numbers.

```bash
$ fdisk -l /dev/sda
Disk /dev/sda: 931.5 GiB, 1000204886016 bytes, 1953525168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: dos
Disk identifier: 0x73c4c07a

# use man fdisk for how to create a new partition.
```

Also you can use  `cfdisk` , which is more humanity.

`UEFI`  mode need a *ESP* partition ，click [here](https://wiki.archlinux.org/index.php/EFI_System_Partition)  for more in details.


```bash
$ mkfs.vfat /dev/sda1
$ mount /dev/sda1 /mnt/boot
```

Format new partitions.

```bash
# /dev/sda1  /
# /dev/sda2  /home
# /dev/sda3  swap
$ mkfs.ext4 /dev/sda1
$ mkfs.ext4 /dev/sda2
$ mkswap /dev/sda3
```

Mount partitions.

```bash
$ swapon /dev/sda3

# Make sure mount / first.
$ mount /dev/sda1/ /mnt
$ mkdir /mnt/home && mount /dev/sda2 /mnt/home

# Check the mount details.
$ lsblk -f
```

Install target system.

```bash
# WIFI network needs these, iw dialog wpa_supplicant wpa_actiond
$ pacstrap /mnt base base-devel
```

Generate fstab to target system.

```bash
$ genfstab -U /mnt >> /mnt/etc/fstab
```

Switch to the target system.

```bash
$ arch-chroot /mnt /bin/bash
```

Set timezone and local time.

```bash
$ ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
$ hwclock --systohc --utc
```

Set language.

```bash
$ locale-gen

$ sed -i '/zh_CN\.UTF\-8/ s/#//' /etc/locale.gen
$ sed -i '/en_US\.UTF\-8/ s/#//' /etc/locale.gen
$ locale-gen

$ cat > /etc/locale.conf << EOF
LANG=en_US.UTF-8
LC_CTYPE=zh_CN.UTF-8
EOF
```

Set hostname and hosts.

```bash
# hostname
$ echo 'JeffArch' > /etc/hostname

# hosts
$ cat > /etc/hosts << EOF
127.0.0.1 localhost
::1       localhost
EOF
```

Set root password.

```bash
$ passwd root
```

Install grub.

```bash
# os-prober uses for dual-system
$ pacman -S grub os-prober
```

Install grub to disk.

```bash
# MBR mode without sda partition number
$ grub-install --target=i386-pc /dev/sda --recheck

# UEFI mode， mount 512M esp partition to /boot
$ pacman -S efibootmgr dosfstools
$ grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub --recheck
```

Generate grub configuration.

```bash
# system can not boot without this action
$ grub-mkconfig -o /boot/grub/grub.cfg
```

Enable dhcp client service.

```bash
$ systemctl enable dhcpcd
```

Exit target system.

```bash
$ exit
```

Umount target system and reboot computer.

```bash
$ umount -R /mnt && reboot
```

Remember to remove usb disk.

## Initial System

> Read this [catalogue](https://wiki.archlinux.org/index.php/General_recommendations) first !

Login as root and create a new normal user for daily management.

```bash
$ useradd -m -G wheel -s /bin/bash jeff
$ passwd jeff
```

Enable `sudo` function to the new user.

```bash
$ echo "%jeff ALL=(ALL:ALL) NOPASSWD:ALL" > /etc/sudoers.d/jeff
$ chmod 600 /etc/sudoers.d/jeff
```

Install extra fonts.

```bash
# google open source fonts
$ sudo pacman -S noto-fonts noto-fonts-cjk noto-fonts-emoji

# wenquanyi
$ sudo pacman -S wqy-microhei wqy-zenhei

$ sudo pacman -S ttf-dejavu

$ sudo pacman -S ttf-win7-fonts ttf-office-2007-fonts

# monaco fonts
```

Install display server，which I chose here is [Xorg](https://wiki.archlinux.org/index.php/Xorg). You can not run desktop environment without it.

```bash
$ sudo pacman -S xorg # choose the default vaules
```

Install graphical driver，vesa is default，others can click  [ATI](https://wiki.archlinux.org/index.php/ATI) 、[Intel](https://wiki.archlinux.org/index.php/Intel_graphics) 、[Nvidia](https://wiki.archlinux.org/index.php/NVIDIA).

```bash
# check what your graphic driver is
$ lspci | grep VGA

$ sudo pacman -S xf86-video-vesa # default
$ sudo pacman -S xf86-video-ati  # AMD
$ sudo pacman -S xf86-video-intel  # intel
$ sudo pacman -S xf86-video-nouveau  # nvidia
$ sudo pacman -S virtualbox-guest-utils # virtualbox
```

Install [Display Manager](https://wiki.archlinux.org/index.php/Display_manager)，here as shown：

1. `gdm` for *GNOME* desktop.
2. `sddm`  for KDE desktop.
3. `lightDM`  for Cross-desktop display manager, like ubuntu unity desktop.

Install [Desktop environment](https://wiki.archlinux.org/index.php/Desktop_environment)，choose one of your faviour.    
*Gnome Desktop*

```bash
$ sudo pacman -S gnome gnome-extras
```

*xfce4 Desktop*

```bash
$ sudo pacman -S xfce4 xfce4-goodies sddm
```

*KDE Desktop*

```bash
$ sudo pacman -S plasma kde-applications-meta sddm kde-l10n-zh_cn
```

*KDE Minimal Desktop*

```bash
$ sudo pacman -S plasma dolphin kate konsole sddm kde-l10n-zh_cn
```

Enable display server auto run when system boot up.

```bash
$ sudo systemctl enable gdm # or sddm or lightDM
```

Install input method, either Ibus or fcitx.

- [Ibus](https://wiki.archlinux.org/index.php/IBus)

```bash
$ sudo pacman -S ibus

$ sudo pacman -S ibus-qt

$ sudo pacman -S ibus-rime

$ ibus-setup

$ cat >> ~/.bashrc << EOF
export GTK_IM_MODULE=ibus
export XMODIFIERS=@im=ibus
export QT_IM_MODULE=ibus
EOF
```

- [Fcitx](https://wiki.archlinux.org/index.php/Fcitx)

```bash
$ sudo pacman -S fcitx fcitx-rime fcitx-configtool fcitx-gtk3 fcitx-gtk2 fcitx-qt5

$ fcitx-config-gtk3

echo -e "GTK_IM_MODULE=fcitx\nQT_IM_MODULE=fcitx\nXMODIFIERS=@im=fcitx" | sudo tee -a /etc/environment
```

Now reboot new system and login it again to the desktop environment.

## Software

Recommend these software as shown here.

Basic software

```bash
$ pacman -S wget zsh w3m curl openssh sudo terminator proxychains vim fakeroot net-tools gparted zip unzip unrar tree ntfs-3g yajl ufw archey3 cronie git
```

Multiply media software

```bash
$ pacman -S gedit remmina libvncserver freerdp smplayer smplayer-themes qbittorrent chromium mkvtoolnix-cli mkvtoolnix-gui aegisub subtitleeditor firefox extra/firefox-i18n-zh-cn telegram-desktop chromium
```

- **SecureCRT**

```bash
# download the tar file yourself
$ tar -xf scrt-8.1.4.1443.ubuntu16-64.tar.gz
$ sudo mv scrt-8.1.4 /usr/local/
$ sudo ln -s /usr/local/scrt-8.1.4/SecureCRT /usr/bin/SecureCRT
$ sudo cp /usr/local/scrt-8.1.4/SecureCRT.desktop /usr/share/application
$ sudo perl securecrt_linux_crack.pl /usr/bin/SecureCRT
```

- **Virutalbox**

```bash
$ sudo pacman -S virtualbox virtualbox-host-dkms
$ sudo /sbin/rcvboxdrv setup
$ sudo gpasswd -a $USER vboxusers
```

- **aria2**

aria2 downloader.

```bash
$ cat > /lib/systemd/system/aria2.service << EOF
[Unit]
Description=Aria2c download manager
After=network.target

[Service]
Type=simple
User=${USER}
ExecStart=/usr/bin/aria2c --conf-path=/home/${USER}/.aria2/aria2.conf

[Install]
WantedBy=multi-user.target
EOF

$ systemctl daemon-reload
```

Create your own *aria2.conf*, you can google it for more in details.

- **LNMP**

Install **nginx**

```bash
$ sudo pacman -S nginx-mainline
```

Web username is http, while ubuntu web username is  www-data.

Install **php**

```bash
$ sudo pacman -S php php-fpm
```

Install **mysql**


Choose mariadb or percona.
```bash
$ sudo pacman -S mariadb
```

Create data directory
```bash
$ mkdir /data/mysql
$ chown mysql:mysql /data/mysql
```

Change some settings in /etc/mysql/my.cnf.
```bash
# enable innodb
sed -i '/innodb_/ s/#//' /etc/mysql/my.cnf

sed -i 's/no\-auto\-rehash/auto\-rehash/g' /etc/mysql/my.cnf
sed -i '/myisam_sort_buffer_size/a bind-address = 0.0.0.0' /etc/mysql/my.cnf

# change to new data directory
sed -i '/myisam_sort_buffer_size/a datadir = /data/mysql' /etc/mysql/my.cnf
```

Initial database.
```bash
mysql_install_db --user=mysql --basedir=/usr --datadir=/data/mysql
```

Start and enable msqld service.
```bash
systemctl start mysqld
systemctl enable mysqld
```

Set mysql root password.
```bash
mysql_secure_installation
```

## AUR

[Arch User Repository](https://wiki.archlinux.org/index.php/Arch_User_Repository)，short as AUR,  provides another way to user install extra software, which is difficult to use. Thanks to `yaourt` , allows user to install AUR software easily like pacman.

 Add *Yaourt* warehouse, option.

```bash
$ echo -e "[archlinuxfr]\nSigLevel = Never\nServer = http://repo.archlinux.fr/\$arch\n" | sudo tee -a /etc/pacman.conf  ```

Requirement.

​```bash
$ sudo pacman -Sy yaourt fakeroot
```

Install personal software.

```bash
$ yaourt -S typora  # markdown editor
$ yaourt -S netease-cloud-music
$ yaourt teamviewer-beta

# gnome themes
$ yaourt -S paper-gtk-theme-git
$ yaourt -S gtk-theme-arc-git
$ pacman -S numix-gtk-theme

# gnome icon
$ yaourt -S paper-icon-theme-git
$ yaourt -S papirus-icon-theme-git
$ yaourt -S ultra-flat-icons-blue
$ yaourt -S ultra-flat-icons-green
$ yaourt -S ultra-flat-icons-orange
$ yaourt -S fcitx-skins
```

## Summary

This post was a draft , ignore it pls 😂