---
layout: post
title: Arch Linux安装与使用
date: 2018-02-07
tags: [Linux]
feature-img: "https://images.pexels.com/photos/34658/pexels-photo.jpg?cs=srgb&dl=blogging-computer-desk-34658.jpg&fm=jpg"
---


久闻 `Arch` 的大名，却老是有种拒绝的心态，说白了就是懒……鉴于 `ubuntu` 最近在内核上作死，默认升级时，给升了个不支持网卡的版本，虽然救活了，但也不想再为此折腾了。遂入坑 `Arch` 。


<!--more-->


> 所有打开的 wiki文档链接，均可在左侧切换 简体中文

## 安装U盘

> 官方 [U盘制作指南](https://wiki.archlinux.org/index.php/USB_flash_installation_media) 

建议使用 `dd` 方式制作

```bash
$ dd bs=4m if=/path/to/archlinux.iso of=/dev/sdx && sync
```

## 安装系统

> 建议先阅读 [官方安装指南](https://wiki.archlinux.org/index.php/Installation_guide)

检查启动模式 UEFI 或 BIOS

```bash
# 有结果为UEFI
$ ls /sys/firmware/efi/efivars 
```

检查网络情况

```bash
$ ping -c 5 www.so.com 
```

使用无线 wifi  `建议使用有线网络`

```bash
 # 按操作连接wifi
$ wifi-menu 
```

如果想远程操作，则开启系统的 *SSH* 服务

```bash
$ systemctl start sshd
```

设置时区与时间同步

```bash
$ timedatectl set-ntp true
$ timedatectl set-timezone Asia/Shanghai
```

使用 [reflector](https://wiki.archlinux.org/index.php/Reflector) 设置镜像源

```bash
# 设置网易镜像，以快速选择
$ cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bak
$ echo "Server = http://mirrors.163.com/archlinux/\$repo/os/\$arch" > /etc/pacman.d/mirrorlist

# 先同步下镜像源
$ pacman -Syy

# 选择中国地区，最近20个更新过的镜像源，挑选最快的12个，保存其中10个到镜像源文件
$ reflector --country 'China' -f 12 -l 20 -n 10 -p https --sort rate --save /etc/pacman.d/mirrorlist
```

接着是 `硬盘分区` 

查看硬盘分区情况

```bash
$ lsblk /dev/sdx
```

使用 `fdisk` 进行分区

> Sectors公式：1024 x 1024 x 1024 / 512 x 你想要的G数，即为最终的 sectors

```bash
# 使用fdisk查看硬盘
$ fdisk -l /dev/sda
Disk /dev/sda: 931.5 GiB, 1000204886016 bytes, 1953525168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: dos
Disk identifier: 0x73c4c07a

# 分区过程略过，自行查找教程
# fdisk操作分区，需要自行计算每个区末尾的　sectors　值
```

使用 `cfdisk` 则能够以直观、人性化的方式进行分区，按提示操作即可。

如果是 `UEFI` 模式，则需要再划分一个 *ESP分区* ，[点我看明细](https://wiki.archlinux.org/index.php/EFI_System_Partition) 。


```bash
# 512M，假设为 sda1，格式化为 FAT32
$ mkfs.vfat /dev/sda1

# 挂载时，则必须为
$ mount /dev/sda1 /mnt/boot
```

格式化分区

```bash
# 假设分区对应关系为
# /dev/sda1  /
# /dev/sda2  /home
# /dev/sda3  swap分区
$ mkfs.ext4 /dev/sda1
$ mkfs.ext4 /dev/sda2
$ mkswap /dev/sda3
```

挂载分区

```bash
# 开启 swap 后续使用
$ swapon /dev/sda3

# 先挂载 根分区
$ mount /dev/sda1/ /mnt
$ mkdir /mnt/home && mount /dev/sda2 /mnt/home

# 查看挂载情况
$ lsblk -f
```

安装目标系统

```bash
# 使用无线的，则装上 iw dialog wpa_supplicant wpa_actiond
$ pacstrap /mnt base base-devel
```

生成文件系统信息表，否则无法切换至新系统

```bash
$ genfstab -U /mnt >> /mnt/etc/fstab
```

切换至新系统

```bash
$ arch-chroot /mnt /bin/bash
```

设置时区与时间

```bash
$ ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
$ hwclock --systohc --utc
```

设置区域语言

```bash
$ locale-gen

# 保留英文与简体中文 UTF8，把开头的注释去掉
$ sed -i '/zh_CN\.UTF\-8/ s/#//' /etc/locale.gen
$ sed -i '/en_US\.UTF\-8/ s/#//' /etc/locale.gen
$ locale-gen

## LANG必须为英文，方便以英文查找日志问题，tty只显示英文
$ cat > /etc/locale.conf << EOF  
LANG=en_US.UTF-8
LC_CTYPE=zh_CN.UTF-8
EOF
```

设置主机名 与 hosts

```bash
# 主机名
$ echo 'JeffArch' > /etc/hostname

# hosts
$ cat > /etc/hosts << EOF
127.0.0.1 localhost
::1       localhost
EOF
```

设置ROOT密码

```bash
$ passwd root
```

安装 grub 软件

```bash
# os-prober 是处理双系统启动
$ pacman -S grub os-prober 
```

安装 grub 到硬盘

```bash
# MBR模式，sda不用带数字
$ grub-install --target=i386-pc /dev/sda --recheck

# UEFI模式， boot分区要挂载为上述划分的512M的那个
$ pacman -S efibootmgr dosfstools
$ grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub --recheck
```

生成 grub 启动表

```bash
# 重要，不生成无法启动
$ grub-mkconfig -o /boot/grub/grub.cfg
```

开启 dhcp 服务，以开机后自动获取IP

```bash
$ systemctl enable dhcpcd
```

退出新系统

```bash
$ exit
```

卸载新系统并重启

```bash
$ umount -R /mnt && reboot
```

记得移除U盘噢！

到时，*arch linux* 系统已经安装完毕。

## 完善新系统

> 强烈建议打开 [建议阅读文档](https://wiki.archlinux.org/index.php/General_recommendations)  辅助理解细节

安装工作还没结束呢，继续干吧!!!

以 root 用户登录新系统，创建管理员用户

```bash
# 指定额外组 wheel
$ useradd -m -G wheel -s /bin/bash jeff
$ passwd jeff
```

开启 `sudo` 权限

```bash
# 方法一：指定编辑程序修改
$ EDITOR=vi visudo
# 去掉 「%wheel ALL=(ALL) ALL」所在行的注释，执行时需要输入密码确认
# 去掉 「%wheel ALL=(ALL) NOPASSWD: ALL」所在行的注释，执行时无须密码确认

# 方法二：sed直接修改
$ sed -i '/NOPASSWD/ s/#//' /etc/sudoers
```

以 管理员 用户登录系统

安装字体

```bash
# 思源字体
$ sudo pacman -S noto-fonts noto-fonts-cjk noto-fonts-emoji
## 思源黑体：adobe-source-han-sans-otc-fonts

# 文泉驿
$ sudo pacman -S wqy-microhei wqy-zenhei

# 官方推荐的 dejavu 英文字体
$ sudo pacman -S ttf-dejavu

# 微软的字体
$ sudo pacman -S ttf-win7-fonts ttf-office-2007-fonts

# 终端等宽字体建议　monaco，须手动额外安装
```

安装输入法，二选一。*iBus* 可在状态栏显示键盘图标，*fcitx* 则在桌面显示输入法状态栏。

> 建议在安装完桌面环境后再安装输入法

- [Ibus](https://wiki.archlinux.org/index.php/IBus)

```bash
$ sudo pacman -S ibus

# 为支持某些Qt应用，但安装的是Qt4，我没安装
$ sudo pacman -S ibus-qt 

# 安装 rime
$ sudo pacman -S ibus-rime

# 初始化
$ ibus-setup

# 如果不能使用，则添加以下三条
$ cat >> ~/.bashrc << EOF
export GTK_IM_MODULE=ibus
export XMODIFIERS=@im=ibus
export QT_IM_MODULE=ibus
EOF
```

- [Fcitx](https://wiki.archlinux.org/index.php/Fcitx) 

```bash
$ sudo pacman -S fcitx fcitx-rime fcitx-configtool fcitx-gtk3 fcitx-gtk2 fcitx-qt5

# 修改fcitx输入设定
$ fcitx-config-gtk3

# 如始终无法显示输入法，则
echo -e "GTK_IM_MODULE=fcitx\nQT_IM_MODULE=fcitx\nXMODIFIERS=@im=fcitx" | sudo tee -a /etc/environment
```



安装 显示服务(display server)，此处选 [Xorg](https://wiki.archlinux.org/index.php/Xorg) 。

> 显示服务 必须得安装，否则无法运行桌面环境

```bash
$ sudo pacman -S xorg # xorg群组，选择默认的即可
```

安装显卡驱动，默认的是 **vesa** 驱动，另外三大厂的，则参考 [ATI](https://wiki.archlinux.org/index.php/ATI) 、[Intel](https://wiki.archlinux.org/index.php/Intel_graphics) 、[Nvidia](https://wiki.archlinux.org/index.php/NVIDIA) 。

```bash
# 查看显卡信息
$ lspci | grep VGA

$ sudo pacman -S xf86-video-vesa # 默认显卡驱动
$ sudo pacman -S xf86-video-ati  # AMD
$ sudo pacman -S xf86-video-intel  # intel
$ sudo pacman -S xf86-video-nouveau  # nvidia
$ sudo pacman -S virtualbox-guest-utils # virtualbox
```

**Xorg** 只提供图形环境的基本框架，完整的用户体验还需要其他组件。

<a href="https://wiki.archlinux.org/index.php/Display_manager_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)">显示管理器</a> (Display Manager)，分好几种，根据不同的桌面环境，目前主流的有：

1. `gdm` ：*GNOME* 显示管理器。
2. `sddm` ：KDE 显示管理器。
3. `lightDM` ：跨桌面的显示管理器，*ubuntu unity* 桌面环境用的是这个。　

<a href="https://wiki.archlinux.org/index.php/Desktop_environment_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)">桌面环境</a> (Desktop environment)，选择其一安装

安装 *gnome*

```bash
# gnome 全家桶，自带 gdm 显示管理器
$ sudo pacman -S gnome gnome-extras
```

安装 *xfce4* 

```bash
# 手动指定显示管理器 sddm
$ sudo pacman -S xfce4 xfce4-goodies sddm
```

安装 *KDE* 

```bash
$ sudo pacman -S plasma kde-applications-meta sddm kde-l10n-zh_cn
```

如果 *KDE* 全家桶你不喜欢，则可以精简安装，以后再自己加

```bash
# 至少包含 窗口管理器，任务栏，终端，文件管理器 和 文本编辑器
$ sudo pacman -S plasma dolphin kate konsole sddm kde-l10n-zh_cn
```

记得设置 显示管理器 开机启动

```bash
$ sudo systemctl enable gdm # or sddm or lightDM
```

好了， 现在可以重启系统试试，应该能登录桌面环境了。

## 安装软件

登录到桌面环境的新系统后，可以安装日常所需软件了。以下部分软件只针对个人使用!!!

安装 **必备基础软件** ，均为命令行。

```bash
$ pacman -S wget zsh w3m curl openssh sudo terminator proxychains vim fakeroot net-tools gparted zip unzip unrar tree ntfs-3g yajl ufw archey3 cronie git
```

安装 **影音、上网、下载等** 软件 

```bash
$ pacman -S gedit remmina libvncserver freerdp smplayer smplayer-themes qbittorrent chromium mkvtoolnix-cli mkvtoolnix-gui aegisub subtitleeditor firefox extra/firefox-i18n-zh-cn 
```

安装其他软件，在 [ubuntu当工作机使用](/2016/08/09/ubuntu-setting/) 文章中有罗列过的。

- **SecureCRT**

使用软件包解码使用即可

```bash
# 下载压缩包
$ tar -xf scrt-8.1.4.1443.ubuntu16-64.tar.gz
$ sudo mv scrt-8.1.4 /usr/local/
$ sudo ln -s /usr/local/scrt-8.1.4/SecureCRT /usr/bin/SecureCRT
$ sudo perl securecrt_linux_crack.pl /usr/bin/SecureCRT
```

可能需要自己创建桌面图标，参考 `/usr/share/application` 目录下其他软件的配置即可。

- **sublime text 3**

添加软件源再安装

```bash
$ curl -O https://download.sublimetext.com/sublimehq-pub.gpg && sudo pacman-key --add sublimehq-pub.gpg && sudo pacman-key --lsign-key 8A8F901A && rm sublimehq-pub.gpg
$ echo -e "\n[sublime-text]\nServer = https://download.sublimetext.com/arch/stable/x86_64" | sudo tee -a /etc/pacman.conf
$ sudo pacman -Syu sublime-text
```

或者使用 *yaourt* 进行安装。

- **calibre**

仍旧是执行命令，给你做好全套

```bash
$ sudo -v && wget -nv -O- [https://download.calibre-ebook.com/linux-installer.py](https://download.calibre-ebook.com/linux-installer.py) | sudo python -c "import sys; main=lambda:sys.stderr.write('Download failed\n'); exec(sys.stdin.read()); main()"
```

- **virutalbox**

```bash
sudo pacman -S virtualbox
sudo /sbin/rcvboxdrv setup # 马上加载模块
sudo pacman -S virtualbox-guest-utils
sudo pacman -S virtualbox-guest-iso
# 添加当前用户到vbox组以使用usb设备
sudo gpasswd -a $USER vboxusers 
```

- **shadowsocks**

很强大，很方便的说，支持启用多配置文件。

```bash
sudo pacman -S shadowsocks
touch /etc/shadowsocks/gce.config  # 创建连接配置
sudo systemctl start shadowsocks@gce
sudo systemctl enable shadowsocks@gce
```

- **aria2**

需要自行添加启动服务配置：`/lib/systemd/system/aria2.service`

```bash
[Unit]
Description=Aria2c download manager
After=network.target

[Service]
Type=simple
User=jeff
ExecStart=/usr/bin/aria2c --conf-path=/home/jeff/.aria2/aria2.conf 

[Install]
WantedBy=multi-user.target
```

关键是配置文件的路径。

- **LNMP**

安装 **nginx**

```bash
sudo pacman -S nginx-mainline
```

其默认配置目录的设置缺乏人性化，可直接照搬 **ubuntu** 下的 *nginx* 配置方式。

web用户为 http，ubuntu的为 www-data。

安装 **php** 

```bash
 # 注意php-fpm.sock路径有所不同
sudo pacman -S php php-fpm 
```

安装 **mysql** 


mysql的实现方式有两种： mariadb 和 percona
```bash
sudo pacman -S mariadb
```

设定数据存储目录为 /data/mysql
```bash
mkdir /data/mysql
chown mysql:mysql /data/mysql
```

更改部分设定 /etc/mysql/my.cnf
```bash
## 启用innodb
sed -i '/innodb_/ s/#//' /etc/mysql/my.cnf  
## 启用自动完成 
sed -i 's/no\-auto\-rehash/auto\-rehash/g' /etc/mysql/my.cnf  
## 设定监听地址
sed -i '/myisam_sort_buffer_size/a bind-address = 0.0.0.0' /etc/mysql/my.cnf
## 设定存储目录
sed -i '/myisam_sort_buffer_size/a datadir = /data/mysql' /etc/mysql/my.cnf
```

初始化 
```bash
mysql_install_db --user=mysql --basedir=/usr --datadir=/data/mysql
```

开启服务
```bash
systemctl start mysqld
systemctl enable mysqld
```

设定 root 密码
```bash
mysql_secure_installation
```

> *mariadb* 中，*mysql* 数据库的 *user* 表，存在 *Password* 字段。

- 其他

`pycharm` 、`enpass` 和 `vs code` 下载解压即可使用。

## AUR 与 Yaourt

*Arch* 还支持 <a href="https://wiki.archlinux.org/index.php/Arch_User_Repository_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)">用户软件仓库</a> (Arch User Repository，AUR) ，使用起来略为蛋疼。

好在还有 `yaourt` 包管理器，可以轻松安装 AUR软件包。

 添加 *Yaourt* 软件库

```bash
$ echo -e "[archlinuxfr]\nSigLevel = Never\nServer = http://repo.archlinux.fr/\$arch\n" | sudo tee -a /etc/pacman.conf
```

更新并安装

```bash
$ sudo pacman -Sy yaourt fakeroot
```

安装个人所需软件

```bash
$ yaourt -S typora  # markdown编辑器
$ yaourt -S google-chrome
$ yaourt -S telegram-desktop
$ yaourt -S vivaldi  # 浏览器
$ yaourt -S keepass # 密码工具
$ yaourt -S netease-cloud-music  # 网易云音乐
$ yaourt teamviewer-beta # teamviewer 13测试版

# gnome 主题
$ yaourt -S paper-gtk-theme-git
$ yaourt -S gtk-theme-arc-git
$ pacman -S numix-gtk-theme

# gnome 图标
$ yaourt -S paper-icon-theme-git
$ yaourt -S papirus-icon-theme-git
$ yaourt -S ultra-flat-icons-blue
$ yaourt -S ultra-flat-icons-green
$ yaourt -S ultra-flat-icons-orange
$ yaourt -S fcitx-skins
```

不加 `-S` 则需要手动选择软件包来安装。 

## 存在问题

从 *ubuntu* 切换至 *arch* 作日常使用，发现几个问题：

> 把 /home /data 独立出来到分区别提有多爽了，重装时简直无痛啊!!!

- **Virtualbox**

安装当前最新版 *5.2.6* ，虚拟系统为 *win7* ，开启 无缝模式「seamless」，win7内窗口黑屏，无法显示，暂时无解。

之前在 *ubuntu 16.04 unity* 桌面环境 时也曾遇到过，降级至 **5.1.32** 版本则没问题。

- **Teamviewer**

teamviewer 13 仅能在 *xrog* 环境中运行，解决方案是 *关闭 Wayland 模式* 

```bash
sed -i '/WaylandEnable/ s/#//' /etc/gdm/custom.conf
```

- **快速切换全屏状态导致回退到登录界面** or **显卡驱动**

这毛病我也不太会用英文描述，通过自己捣鼓出来解决了。 

`问题一`：无论什么应用程序，在 *gnome* 桌面环境下，快速切换全屏与还原，屏幕显示出现跳帧，接着退回到登录界面，使用核显 `Intel HD 530` 。

**解决方法**： 卸载 *intel显卡驱动* ， `sudo pacman -Rs xf86-video-intel` ，重启系统即可。



`问题二`： *smplayer* 快速切换全屏与还原，上述问题依旧出现，其他应用则正常，猜测是 **mplayer** 的问题。

**解决方法**： 在 *smplayer* 中依次，**首选项** => **界面** => **高DPI** => **启用高DPI支持** 。 



`问题三`： 使用 *teamviewer* 远程连接桌面时，屏幕隔几秒就闪动，有点跳帧的感觉，影响实际操作。

**解决方法**： 与 **问题一** 相同，卸载 intel 的驱动即可。

## 尾巴

好了，终于写到尾巴了……

回头可以写安装脚本了，再装新系统时，只需要敲个回车执行脚本就可以坐等了，嘿嘿！

关于 天朝特色软件 QQ，可参考 <a href="https://wiki.archlinux.org/index.php/Tencent_QQ_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)">这里</a> ，我个人倾向于使用 *Vitualbox* ，详见<a href="https://wiki.archlinux.org/index.php/VirtualBox_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)">这里</a>。

关于定制 Gnome，需要关注两个网站：一是 [主题网站](https://www.gnome-look.org) ， 二是 [扩展网站](https://extensions.gnome.org) 。个人推荐 [vimix-gtk-themes](https://github.com/vinceliuice/vimix-gtk-themes) 主题。

关于 KDE 的安装与使用，可以参考这个视频：[Plasma 5 on Arch Linux](https://www.youtube.com/watch?v=lv5CyzsIjJk&index=5&list=PLSmXPSsgkZLvh3gR-5koEahEMs3BwOLgN) 。

关于 **Arch Linux** ，对其简洁的哲学深表赞同，滚动升级的方式贼爽，最牛逼的是官方wiki好强大，文档写得有条理，对可能涉及的地方都有详细的解释，啧啧！！！

最后，Arch 放弃32位了，`万恶的steam` 用不了啊?。

基于 *Arch* 的发行版，推荐 [KaOS](https://kaosx.us/) 和 [Manjaro](https://manjaro.org/)，两者均是图形化界面安装，支持 *Steam*；但前者专注 **KDE**，后者专注 `驱动` 。 



### 更新
> 2018-05-08 添加新的推荐主题 [vimix-gtk-themes](https://github.com/vinceliuice/vimix-gtk-themes)
> 2018-05-14 添加推荐的发行版