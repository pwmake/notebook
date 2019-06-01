---
title: ubuntu16使用linksys ae6000
layout: post
date: 2017-04-09 15:58:13
tags: [Skill]
feature-img: "assets/img/pexels/startrails.jpg"
---

*linksys AE6000* 无线网卡在官方无linux驱动的前提下，如何实现安装驱动呢？本人懒得拉线，亦不想闲置无线网卡不用，更别提另外买一个默认支持的无线网卡。前者的网卡芯片是 `mt7610u` ， *github*  上有大神放出其驱动，但最高仅支持 4.7 内核使用，而 *技嘉B250-HD3* 这块板子的蛋疼之处，在于无法安装旧版默认内核为 4.4 的 16.04镜像，蛋疼啊……怎么办呢？

<!--more-->

莫慌，先理清思路，既然该驱动仅支持 4.7 的内核，那就降级内核，再编译驱动进行安装咯！

## ubuntu 16.04.02 内核降级

查询当前可安装的内核版本

```shell
$ sudo apt list | grep linux-headers-4 | grep generic
# linux-headers-4.4.0-72-generic/xenial-updates,xenial-security,now 4.4.0-72.93 amd64 [已安装]
# linux-headers-4.8.0-36-generic/xenial-updates,xenial-security,now 4.8.0-36.36~16.04.1 amd64 [已安装，自动]
```

安装4.4的内核，16.04.02默认的内核版本是 *4.8.0-36* ，此处安装最后一个版本的 4.4内核

```shell
$ sudo apt-get install linux-image-extra-4.4.0-72-generic
```

检查是否安装成功

```shell
$ sudo dpkg -l | grep 4.4.0.72-generic
```

## 调整grub2启动项

我照着文末参考链接处调整grub2内核的启动顺序均以失败告终，直到阅读了官方文档之后才卸下心中的大石。网络上现有的方法均是旧版的，谨慎参考之。

查看grub2默认的启动顺序，可查询文件 `/boot/grub/grub.cfg` ， 从以下注释开始查看

```shell
### BEGIN /etc/grub.d/10_linux ###
```

如果要修改启动顺序，官方则建议修改文件 `/etc/default/grub` ，*ubuntu 16.04* 启动过程中默认不显示 grub2 菜单了，如果需要显示，可修改如下配置；或者在启动过程中，按着 `shift` 键。

```shell
$ sudo sed -i '/GRUB_HIDDEN_TIMEOUT=0/ s/^/\#/g' /etc/default/grub
```

修改完毕后，执行以下命令来更新启动菜单的顺序

```shell
$ sudo update-grub
```

再次查看 */boot/grub/grub.cfg* 以确认是否修改成功。

安装好 *4.4内核* 后，不建议把 *4.8内核* 删除，而是设置默认以 *4.4内核* 启动系统即可。而 *grub2官方文档* 推荐的方法如下

```shell
# 建立自定义的grub配置，有固定的配置文件格式，必须以06_开头命名才能调整为首位
$ sudo cp /etc/grub.d/04_custom /etc/grub.d/06_YourCustomName

# 从现有grub.cfg中查询4.4内核的设置
$ grep "menuentry 'Ubuntu，Linux 4.4.0-72-generic'" /boot/grub/grub.cfg 
# 进入grub.cfg文件，把上述名字的代码块拷贝到自定义配置中

# 修改全员可执行权限
$ sudo chmod a+x /etc/grub.d/06_YourCustomName

# 更新
$ sudo update-grub

# /boot/grub/grub.cfg 中检查是否变更成功
# 只要注释　### BEGIN /etc/grub.d/06_YourCustomName ###　
# 先于 注释　### BEGIN /etc/grub.d/10_linux ###　即代表修改成功
```

最后是贴个完整的自定义grub配置 `06_YourCustomName` 

```shell
#!/bin/sh
exec tail -n +3 $0
# This file provides an easy way to add custom menu entries.  Simply type the
# menu entries you want to add after this comment.  Be careful not to change
# the 'exec tail' line above.
menuentry 'Ubuntu，Linux 4.4.0-72-generic' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-4.4.0-72-generic-advanced-917d9fff-cf63-4334-be89-9a8852628821' {
	recordfail
	load_video
	gfxmode $linux_gfx_mode
	insmod gzio
	if [ x$grub_platform = xxen ]; then insmod xzio; insmod lzopio; fi
	insmod part_gpt
	insmod ext2
	set root='hd0,gpt2'
	if [ x$feature_platform_search_hint = xy ]; then
	  search --no-floppy --fs-uuid --set=root --hint-bios=hd0,gpt2 --hint-efi=hd0,gpt2 --hint-baremetal=ahci0,gpt2  917d9fff-cf63-4334-be89-9a8852628821
	else
	  search --no-floppy --fs-uuid --set=root 917d9fff-cf63-4334-be89-9a8852628821
	fi
	echo	'载入 Linux 4.4.0-72-generic ...'
        linux	/boot/vmlinuz-4.4.0-72-generic root=UUID=917d9fff-cf63-4334-be89-9a8852628821 ro  quiet splash $vt_handoff
	echo	'载入初始化内存盘...'
	initrd	/boot/initrd.img-4.4.0-72-generic
}
```

## AE6000 驱动安装

启动顺序调整好后，重启系统，检查内核是否为 4.4 ，确认无误后，即可开始安装无线驱动了

下载大神的代码

```shell
$ git clone https://github.com/xtknight/mt7610u-linksys-ae6000-wifi-fixes
```

安装头文件

```shell
$ sudo apt-get install build-essential linux-headers-$(uname -r)
```

编译与安装

```shell
$ cd cd mt7610u-linksys-ae6000-wifi-fixes
$ sudo make clean && sudo make && sudo install
```

编译DKMS

```shell
$ sudo apt install dkms
$ sudo cp -R . /usr/src/mt7610u_sta-1.0
$ sudo dkms add mt7610u_sta/1.0
$ sudo dkms build mt7610u_sta/1.0
$ sudo dkms install mt7610u_sta/1.0
```

重启系统，登录后，已经显示可连接的WIFI，记住，该驱动 `仅支持2.4G wifi信号` ，选5G的话，连接失败，必须重启才能再次连接2.4G的。

## 尾巴

为了实现Linux系统中显卡与无线网卡正常驱动，搞死俺了。期间还折腾了一下 *debian 8.7.1* ，嘿嘿！还是用回 *ubuntu 16.04* 顺手啊，哈哈！

## 参考链接

> [内核降级](http://www.xf5000.com/2016/12/20/ubuntu16-04-kernel-downgrade/)
>
> [Grub2使用说明](https://wiki.ubuntu-tw.org/index.php?title=Grub2)
>
> [AE6000 WIFI 驱动](https://github.com/xtknight/mt7610u-linksys-ae6000-wifi-fixes)