---
title: 技嘉B250安装Debian 8.7.1
layout: post
date: 2017-04-07 11:02:37.387000
tags: [Skill, Linux]
feature-img: "assets/img/pexels/startrails.jpg"
---


最近装了个主机，200系列的主板只支持 *window8/10* 系统，*window7* 默认不支持了，网上有教程指导如何破解主板限制安装win7，这尼玛看着都 有点尴尬了。考虑到在win下部署开发环境略为蛋疼，遂打算安装个 *linux* 系统作日常使用，岂料一波三折啊。

<!--more-->

## 制作启动U盘

现今的主板均已支持UEFI，*linux* 系统镜像亦是如此。试用过如下方案：

1. ubuntu 16.04 amd64镜像，Rufus 工具刷入，进入 *install ubuntu 16.04* 界面后黑屏，失败
2. linux mint 18 64位镜像，同上刷入，同上失败
3. ubuntu 16.04.02 镜像，同上刷入，选择 *mbr分区表的BIOS或UEFI* ，成功安装

## 无线网卡驱动

以上述第三种方案安装完并成功进入系统后，需要安装无线网卡驱动。正在使用的是 *linksys ae6000* 无线网卡，网上有大神制作的驱动程序，欣喜啊，嘿嘿。阅读完使用说明后，心凉了一截，最高仅支持 4.7的内核。尝试把 *ubuntu 16.04.02* 进行降为 4.4 内核，额……驱动爆炸啊……哎……。我又不想拉条网线插主机上，好麻烦……

## 安装Debian 8.7.1

来来来，正式进入主题了。安装这玩意步骤甚多，略为麻烦。初次安装*debian* 的我有点不知所措。

制作启动U盘，直接使用命令行，前提是要卸载U盘但不要推出

```shell
sudo dd if=debian-8.7.1-amd64-CD-1.iso of=/dev/disk2 bs=1m
```

*if* 与 *of* 分别代表镜像文件 与 U盘的实际路径。

我使用的是CD版的镜像，因此安装过程中需要连接网络，否则……很蛋疼……过程中检查网卡时，报缺少网卡驱动 *rtl8168g-2.fw，是否在可移动设备即U盘中查找* 。

解决方法是

1. 下载包 firmware-realtek_0.43_all.deb
2. 使用归档工具解压，找到lib/firmare文件夹，拷贝到第二个FAT格式的U盘
3. 把第二个U盘插入到主机中
4. 在安装界面中，选择退回，选择执行一个命令行窗口 *excute a shell*

```shell
# 查找第二个U盘的名字
ls /dev/sd* 
# 借助分区表来识别U盘名字，假设为/dev/sdd2
fdisk -l 
# 把第二个U盘的firmware文件夹拷贝到根目录 / 下，或者直接把rtl8168g-2.fw拷贝到根目录下
mount /dev/sdd2 /media
cp -ap /media/firmware /
```

1. 退出命令行窗口，重新执行安装操作即可。

接着安装过程中问你是否连接到镜像站点来更新软件，网络状态好的选择是，否则选NO吧。要等好久的说……

装完重启，直接进入命令行，无图形化界面，以上选择NO的话默认没安装，执行安装之前先更新下国内的[软件源](http://mirrors.163.com/.help/debian.html)，提升下载速度。命令行浏览网站可以使用 `w3m` 。

```shell
# 备份旧源
cp /etc/apt/sources.list /etc/apt/source.list.bak
# 下载国内源
cat > /etc/apt/sources.list << EOF
deb http://mirrors.163.com/debian/ jessie main non-free contrib
deb http://mirrors.163.com/debian/ jessie-updates main non-free contrib
deb http://mirrors.163.com/debian/ jessie-backports main non-free contrib
deb-src http://mirrors.163.com/debian/ jessie main non-free contrib
deb-src http://mirrors.163.com/debian/ jessie-updates main non-free contrib
deb-src http://mirrors.163.com/debian/ jessie-backports main non-free contrib
deb http://mirrors.163.com/debian-security/ jessie/updates main non-free contrib
deb-src http://mirrors.163.com/debian-security/ jessie/updates main non-free contrib
EOF
```

安装图形化界面，有三种界面可选，分别是 *gnome* 、*kde* 、*lxde* ，容量从大到小排序

```shell
apt-get install xorg
# gnome界面
apt-get install gnome
# kde界面
apt-get install kde
# lxde
apt-get install lxde
```

安装无线网卡驱动，重启后即可使用，只能用2.4G的网络。

## 参考链接

> [Debian官方安装说明，有谈分区设定](https://www.debian.org/releases/stable/i386/ch06s03.html.zh-cn#di-partition)
>
> [网易debian软件源](http://mirrors.163.com/.help/debian.html)
>
> [rtl8168g-2.fw网卡驱动包](https://packages.debian.org/search?searchon=contents&keywords=rtl8168g-2.fw) 
>
> [linksys ae6000 无线网卡驱动](https://github.com/xtknight/mt7610u-linksys-ae6000-wifi-fixes)