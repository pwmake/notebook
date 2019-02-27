---
title: win10硬盘安装ubuntu
layout: post
date: 2016-08-06 00:00:00
tags: [Skill]
feature-img: "https://images.pexels.com/photos/109371/pexels-photo-109371.jpeg?cs=srgb&dl=computer-internet-keyboard-109371.jpg&fm=jpg"
---


心血来潮打算在现有win10环境下安装个ubuntu系统，以便用于网站开发，毕竟windows环境中布署真心很蛋痛。很久没折腾过了，依稀记得把ubuntu镜像借用工具写入到U盘即可；可认真实践起来……问题不断……

<!--more-->

下面是详细的折腾过程，win10为主系统，分区表是`MBR`格式，安装的是*ubuntu 16.04* 。


## U盘安装

- 使用*UltraISO* 写入到U盘

失效，无法写入完整的文件

- 使用*Universal_USB* 写入到U盘

写入成功，重启电脑以U盘启动，安装过程到分完区时，系统提示无法找到安装源，即U盘识别有问题，无法继续安装。

重试+1

重试+2

放弃之……排除U盘有问题

## 硬盘安装

- 安装*EasyBCD*

安装后运行软件，依次点击：`添加新条目`=>`NeoGrub`=>`安装`

- 配置引导程序

点击安装后，再点击`配置` ，编辑配置文件`menu.lst`，粘贴以下内容

```shell
# 标题随便自定
title Install Ubuntu 16.04
# 指定原来引导所在的分区，通常是C盘
root (hd0,0)
# 指定ubuntu启动文件与镜像文件
kernel (hd0,0)/vmlinuz boot=casper iso-scan/filename=ubuntu16.iso ro quiet splash locale=zh_CN.UTF-8
initrd (hd0,0)/initrd.lz
```

- 准备文件

1. 把ubutn镜像重命名为*ubuntu16.iso*并拷贝到C盘根目录
2. win10中右键镜像文件，点选加载，把里头的`casper/vmlinuz`和`casper/initrd.lz` 拷贝到C盘根目录

前方高能预警，这里很重要

```shell
# 32位镜像
vmlinuz
# 64位镜像
vmlinuz.efi
```

请根据你要安装的ubuntu版本来选择，并相应更改*menu.lst* 中的内容

重启进入安装界面，打开终端`ctrl+alt+t`，输入

```shell
# 防止安装过程中出现 “无法卸载挂载点” 的错误
sudo umount -l /isodevice
```

- 引导进入ubuntu系统

重启进入win10，运行*easyBCD* ，依次下列操作

1. `添加新条目`=>`NeoGrub`=>`删除`
2. `添加新条目`=>`Linux/BSD`

```
类型：	GRUB（Legacy)
名称： 自定
驱动器： 选择/boot 或 / 所在分区
```

确认 添加条目 即可

最后把刚才拷贝到C盘的文件都删除掉吧