---
layout: post
title: grub4dos制作多合一启动U盘
tags: [Skill]
date: 2017-05-24
feature-img: "https://images.pexels.com/photos/229543/pexels-photo-229543.jpeg?cs=srgb&dl=close-up-connector-macro-229543.jpg&fm=jpg"
---

工作中需要重装系统，老是需要格盘以安装不同的系统，略为蛋疼。遂打制作多合一启动U盘，4G容量足矣，专用于重装系统。

<!--more-->

## 格式化U盘

使用 `Bootice` 软件操作，顺序依次为

`物理磁盘` => `目标磁盘` => `主引导记录` => `GRUB4DOS (grldr.mbr)`  => `安装/配置` 。

此法的分区表为 **MBR** 格式，若要 **GPT** 格式的，可参考末尾链接。

下载 *grub4dos* 压缩包并解压之，拷贝 `grldr` 到U盘根目录。

## 制作启动菜单

在Ｕ盘根目录，新建 `menu.lst` ，贴入以下内容

```shell
color blue/green yellow/red white/red white/red
timeout 30
find --set-root --ignore-floppies --ignore-cd /grldr
## menu border color
color border=0xEEFFEE
## set vbe mode
graphicsmode -1 640:800 480:600 24:32 || graphicsmode -1 -1 -1 24:32
## 加载启动时的背景图 800x600 image size
splashimage /splash.bmp
default /default
## Menu AutoNumber
write 0x8274 0x2001

title XiaoMa 2013 PE 
find --set-root --ignore-floppies --ignore-cd /XMPE2013.ISO
map /XMPE2013.ISO (0xff)
map --hook
chainloader (0xff)
savedefault --wait=2

title CDLinux-0.9.6.1
find --set-root --ignore-floppies --ignore-cd /CDlinux/bzImage
kernel /CDlinux/bzImage CDL_DEV=LABEL=CDlinux CDL_LANG=zh_CN.UTF-8
initrd /CDlinux/initrd
boot

title lubuntu-16.04.3-desktop-amd64
find --set-root --ignore-floppies --ignore-cd /lubuntu-16.04.3-desktop-amd64.iso
map /lubuntu-16.04.3-desktop-amd64.iso (0xff)
map --hook
chainloader (0xff)
kernel /lubuntu/casper/vmlinuz.efi boot=casper iso-scan/filename=/lubuntu-16.04.3-desktop-amd64.iso noprompt noeject
initrd /lubuntu/casper/initrd.lz
savedefault --wait=2

title ubuntu-16.04.3-server-amd64 
find --set-root --ignore-floppies --ignore-cd /ubuntu-16.04.3-server-amd64.iso
map /ubuntu-16.04.3-server-amd64.iso (0xff)
map --hook
chainloader (0xff)
savedefault --wait=2

title reboot (重启)
reboot

title halt (关机)
halt

# 正确显示中文
5173:10100810082000003FF8010001000100FFFE010002800280044008203018C006
542F:010000801FFC1004100410041FFC10001000100017FC24042404440487FC0404
673A:100011F011101110FD10111031103910551055109110111211121212120E1400
91CD:001000F83F000100FFFE01001FF011101FF011101FF001003FF80100FFFE0000
```

其中，**Lubuntu** 的启动需要提取镜像文件中的 `casper` 出来方能正常引导。

**CDLinux** 则需要解压整个镜像出来，直接放在根目录。

背景底图，格式必须为 *.bmp* ，大小为 **800 x 600** ，24位色。

U盘的目录结构为

```shell
.
├── CDlinux
│   ├── boot
│   ├── bzImage
│   ├── doc
│   ├── extra
│   ├── initrd
│   ├── lang
│   ├── local
│   └── settings
├── grldr
├── lubuntu
│   └── casper
│       ├── initrd.lz
│       └── vmlinuz.efi
├── lubuntu-16.04.3-desktop-amd64.iso
├── menu.lst
├── splash.bmp
├── ubuntu-16.04.3-server-amd64.iso
└── XMPE2013.ISO
```

搞定！

## 参考链接

> [Bootice 启动维护工具](http://www.ipauly.com/category/works/)
>
> [grub4dos 启动引导管理器](http://grub4dos.chenall.net/)
>
> [支持 BIOS 与 UEFI 的U盘引导方案](https://my.oschina.net/abcfy2/blog/491140)