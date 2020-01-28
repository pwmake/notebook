---
layout: post
title: LEDE/OpenWRT使用概览
date: 2018-05-01
tags: [Reference]
feature-img: "https://images.pexels.com/photos/159304/network-cable-ethernet-computer-159304.jpeg?cs=srgb&dl=cables-close-up-connection-159304.jpg&fm=jpg"
---

业务需要，尝试调研 `LEDE/OpenWRT`，如今这两者合并了，详细的介绍请参阅 [LEDE维基百科](https://zh.wikipedia.org/wiki/LEDE)。


<!--more-->


花费一周的晚上时间来尝试了解 **OpenWRT** ，鉴于部分网络知识遗忘得差不多，花了点时间补充回来，使得后续阅读能够继续进行。本文借助 `O'Reilly` 技术类书籍的逻辑思路来说明，即：`question, solution, disscusion`。

> 务必打开 [LEDE 用户手册中文导航](https://openwrt.org/zh/docs/user-guide/start) 来同步阅读！以下简称为 `手册导航`

## 下载与安装

`Question`：如何在路由器安装该固件呢？

`Solution`： 手册导航中 **新手部分** 已有明确的路由器下载与安装指导，想快速查看自己的路由器是否支持，可查询 [硬件列表](https://openwrt.org/toh/views/toh_fwdownload) 。

`Disscusion`：如果想先在虚拟机中体验一把，可使用 **Virtualbox** 安装，官方文档是 [将LEDE作为Virtualbox虚拟机运行](https://openwrt.org/docs/guide-user/virtualization/virtualbox-vm)，但此法需要手动做镜像转换，个人认为较为麻烦，在下的做法是：

1. 新建vbox虚拟机，选 `linux 其他 64位 2.6/3.x/4.x` 版本
2. 挂载任意 **Ubuntu** 镜像，以进入 Live 系统
3. 从固件列表中下载任意 `x86` 固件
4. 在终端解压固件：`sudo gzip -d *.img.gz`
4. 在终端中执行： `sudo dd if=固件名.img of=/dev/sda`
5. 重启虚拟机即可 


## 使用与配置

`Question`： 如何对路由器进行管理呢？

`Solution`： 网线连接路由器后，访问 `192.168.1.1` 即可，默认密码为空。虚拟机方案的，则有点复杂。继续上述虚拟机方案，则需要

1. LEDE所在虚拟机使用双网卡，网卡1使用 `内部网络`，并命名为 **wrt**，网卡2为桥接网络
2. 另外再新建任意虚拟机，网卡使用同样使用 内部网络，名字同为 **wrt**
3. 在新虚拟机浏览器中，访问 *192.168.1.1* 即可

`Disscusion`：手册导航中，有 **基本配置** 这一节，但部分设定是否看得很懵逼？是时候该补一波了！首先是基本网络知识，如OSI七层、TCP/IP四层、IP、子网掩码、端口，可参考 [Linux网络管理](http://www.imooc.com/learn/258)；其次是路由相关，如静态路由、动态路由、NAT等等，可参考 [CCNA教程](https://www.youtube.com/playlist?list=PLsG5PCdy5Bd4-KZpm8qV3jtA1QKibrXNI)。网关、路由与交换机三者也得区分清楚，建议使用虚拟机环境来实践，会有更好的理解，补充两个阅读项：[Linux 网络接口](https://openwrt.org/zh-cn/doc/networking/network.interfaces) 与 [交换机手册](https://openwrt.org/zh-cn/doc/uci/network/switch)。

> 建议使用 SSH 连接到LEDE所在虚拟机，对网络做任何改动时，随时查看相应配置文件，强化原理学习。

## 怎么玩？玩什么？

`Question`：手册导航中的 **基本配置** 也了解完了，接下来可以做些啥？

`Solution`： 继续参考 手册导航 中的添加存储设备，以及额外服务呗。又或者是安装 [koolshare LEDE](http://firmware.koolshare.cn/LEDE_X64_fw867/)固件，正所谓 `模仿是最好的学习`，看看别人都额外添加了哪些内容，自己能否也添加上去。`酷软` 一栏的内容则不必参考了，那些是闭源的东东，shell脚本也做过加密处理了。 

`Disscusion`： 那该怎样实现 **墙墙墙** 呢？仓库源中默认带了一个 `SS`，但不支持[之前文章](/2018/04/09/v2ray-usage-summary)所说的 `AEAD`，至于如何实现 `透明代理`，则参考 [那些从墙上学会的知识](https://icymind.com/learn-from-gfw/)，该楼主另一篇文章也可以阅读下噢，在 [这里](https://icymind.com/virtual-openwrt/)。

## 编译固件

`Question`： 我想要编译自己的固件，怎么弄？

`Solution`： 手册导航中有 [初学者构建自己的固件的分步指南](https://openwrt.org/docs/guide-user/additional-software/beginners-build-guide) 可作参考，初次编译时，如何选择正确的 `Target` 呢？官方文档中描述的不是太明确。可在 硬件列表 中快速查找，表格中有 `Target`「对应CPU类型」、`Subtarget` 字段，`target profile`则对应的是路由器具体型号名称。编译选择中有三种状态，M状态表示不包含在固件中，可后续添加上去安装。

`Disscusion`： 编译固件之前，先要明确为何要这样做？如果纯粹是为了减少手动安装软件包的麻烦，倒不如直接写个shell脚本处理即可。出于额外添加了自定义的软件包，为了大批量分享给他人使用的情况下才会考虑编译固件，如不是特别明确自身的需求，编译固件是为可选项。

## 小结

本文大概罗列了 `LEDE` 使用过程中4大问题，仅是粗略的说明，具体每个使用选项，需要自行深入研究，时间有限，仅此而已。
