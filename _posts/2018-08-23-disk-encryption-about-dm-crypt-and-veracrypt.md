---
layout: post
title: VeraCrypt 与 dm-crypt 磁盘加密
date: 2018-08-23
tags: [HowTO]
feature-img: https://images.pexels.com/photos/721480/pexels-photo-721480.jpeg?cs=srgb&dl=blur-cabinet-door-721480.jpg&fm=jpg
---


工作中需要定期备份数据，对移动硬盘进行加密处理，脑海中首选N年前在 `编程君` 网站上学习过的 `TrueCrypt` ，免费、开源、多平台，还带 `演戏模式` ，啧啧。`TrueCrypt` 因安全问题中断开发多年，代替者为 `VeraCrypt` 。

<!--more-->

## 文件系统格式

先来了解下现在主流的 **文件系统格式** ：

- `FAT/FAT32` ：微软很早之前的文件系统，无权限管理，并且有单分区最大文件不得超过 `4G` 的限制。
- `exFat` ：微软继 `fat32` 而推出的，单文件大小限制升级到 `12G` 「不记得确切数据」，mac系统也支持。
- `ntfs` ：还是微软的，这回带了权限管理，系统耐操了……
- `EXT4` ： linux 主流文件系统格式
- `XFS` ：为适应新时代大数据容量盘，直观感受是格式时长远低于 `EXT4` 。

详细的区别需要自行谷歌之。

某些 Linux 系统，需要安装以下软件包 ，方可使用 加密

```shell
$ apt install fuse fuse-devel
```

## 原生加密

Linux 原生自带了一个磁盘加密软件 ——  `cryptsetup` ，详细请参考 [使用 LUKS 硬盘加密](https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/7/html/security_guide/sec-encryption) 。主要分为 `物理加密` 与 `虚拟加密` ，支持指定 `Key File` 、`迭代时间` 的设定。

查看版本， 1.6 及以上

```shell
$ cryptsetup --version
```

查看性能指标

```shell
$ cryptsetup benchmark
```

打开加密盘

```shell
$ cryptsetup open --type 类型名 已加密的物理设备或逻辑设备 映射名
```

查看加密盘状态

```shell
$ cryptsetup status 映射名
```

关闭加密盘

```shell
$ cryptsetup close 映射名
```

举个栗子，创建一个 `虚拟加密盘` ，挂载，使用，接着卸载。

```shell
# 创建 1G 一个文件作为容器
$ dd if=/dev/zero of=/root/luks.vol bs=1M count=1024
# 或者
$ fallocate -l 64G /root/luks.vol  # 此法瞬间完成

# 使用 LUKS 方式加密「格式化」该文件容器
$ cryptsetup --cipher aes-xts-plain64 --key-size 512 --hash sha512 --iter-time 10000 luksFormat /root/luks.vol
password: # 设置密码

# 打开加密后的文件容器
$ cryptsetup open --type luks /root/luks.vol fuckme
# 或者
$ cryptsetup luksOpen /root/luks.vol fuckme
# 查看映射情况
$ ls /dev/mapper
fuckme

# 挂载
$ mount /dev/mapper/fuckme /mnt

# 使用完后卸载
$ umount /mnt
$ cryptsetup close fuckme
```

`物理加密` 同理。

## veracrypt

详细说明参考 [扫盲 veraCrypt](https://program-think.blogspot.com/2015/10/VeraCrypt.html) 。刚上手使用，建议在 `图形界面` 中创建加密盘。

Linux 下的安装，到 [官网](https://www.veracrypt.fr/en/Downloads.html) 下载软件包，解压至系统

```shell
$ tar -xf veracrypt-1.22-setup.tar.bz2

# 根据对应平台选择，此处选命令行 x64
$ ./veracrypt-1.22-setup-console-x64
VeraCrypt 1.22 Setup
____________________


Installation options:

 1) Install veracrypt_1.22_console_amd64.tar.gz
  2) Extract package file veracrypt_1.22_console_amd64.tar.gz and place it to /tmp

  To select, enter 1 or 2: 1

  省略N行...
  Do you accept and agree to be bound by the license terms? (yes/no): yes

# 检查是否安装成功
$ which veracrypt
/bin/veracrypt
```

`VeraCrypt` 略显智能，当你打开加密盘时，输入密码后，它会自动帮你挂载之。如果系统没有相应的文件系统格式支持，则报无法挂载，或者是挂载为只读。其支持 `4种` 文件系统格式：`FAT` ，`exFat` ，`NTFS` 和 `none` 。

安装 `exFat` 和 `NTFS` 支持

```shell
# exfat for centos 7
$ yum install -y http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-1.el7.nux.noarch.rpm
$ yum install exfat-utils fuse-exfat

# exfat for ubuntu 16.04
$ apt install exfat-fuse exfat-utils

# ntfs 
$ yum/apt install ntfs-3g ntfs-3g-utils 
```

同样举个 栗子 辅助说明，从挂载，使用，再到 卸载。

```shell
# veracrypt 加密文件 或者 加密分区  待挂载的目录

# 挂载 test02 这个虚拟加密盘 到 /mnt
$ veracrypt /tmp/test02 /mnt
Enter password for /tmp/test02: 
Enter PIM for /tmp/test02: 
Enter keyfile [none]: 
Protect hidden volume (if any)? (y=Yes/n=No) [No]: 

# 查看目录是否有挂载后的文件
$ ls /mnt

# 检查映射情况
$ ls /dev/mapper
veracrypt1

# 查询当前打开的加密盘
$ veracrypt -l
1: /tmp/test02 /dev/mapper/veracrypt1 /mnt

# 使用完后
$ umount /mnt
$ veraycrypt -d /tmp/test02

# 检查是否成功卸载加密盘
$ veracrypt -l
Error: No volumes mounted.
```

在 **CentOS 7** 中，挂载 `exFat` 格式的加密盘无效，即使 安装了 *exfat* 的软件包，暂时未知。在 `Ubuntu 16.04` 中正常。可以使用下节的方法挂载之。

## 两者关系

有了上述的知识为基础，务必阅读完 [扫盲 dm-crypt](https://program-think.blogspot.com/2015/10/dm-crypt-cryptsetup.html) ，里面很多详细的玩法说明讲解。

这里举个栗子，使用 *cryptsetup* 来挂载 `veracrypt` 的加密盘。

```shell
# 打开
$ cryptsetup open --type tcrypt --veracrypt ntfs-test ntfs

# 根据映射名挂载
$ mount /dev/mapper/ntfs /mnt/
# 如果是 exfat 格式
$ mount -t exfat /dev/mapper/ntfs /mnt/

# 卸载
$ umount /mnt/

# 关闭
$ cryptsetup close ntfs
```

## 小结

如无特殊需求，仅使用密码即可。

虚拟加密盘，适用加密文件，再丢个网盘同步，专治流氓。

物理加密盘，切记不要轻易格盘，否则数据全丢，务必放置合适的地方。


