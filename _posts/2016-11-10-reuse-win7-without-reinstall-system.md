---
title: win7免装系统更换主板
layout: post
date: 2016-11-10 00:00:00
tags: [Skill]
feature-img: "assets/img/pexels/startrails.jpg"
---


工作的缘故，需要对某台主机进行迁移，直接把硬盘给换到别的主机上使用，岂料开机后无法进入系统，禁用驱动，卸载了旧有的驱动也无法启动。后来网上搜索答案，结果获知原来系统默认自带一个神器——`sysprep` ，可以让你迁移硬盘时能够避免重装系统。

<!--more-->


进入待迁移的操作系统，运行

```shell
cd C:\windows\system32\sysprep
sysprep.exe
# 选上OOBE、勾选Generalize(通用)
# 选关机
# 把硬盘迁移至新主机，开机，初始化硬件信息，建立新用户
# 重启后以老用户进入系统，删除新用户，安装驱动即可
```

按上述步骤操作时，重新初始化硬件信息时报错

```shell
安装程序正在为首次使用计算机做准备	
```

弹出的对话框显示

```bat
计算机意外地重新启动或遇到错误，windows安装无法继续，若要安装windows，请单击“确定”，重新启动计算机，然后重新启动安装
```

解决的办法是

```shell
# 同时按住 shift + F10打开命令pwkk
cd c:\windows\system32\oobe
msoobe.exe
# 继续按照正常步骤安装，无视上述弹出的对话框内容
```

神奇的BUG

以新用户登录系统后，启用 *administrator* 用户，并且把 *新用户* 删除掉，重启系统后，居然又是以 *新用户*  登录，这尼玛就神奇了，还能还原回去……目前仍然无解中……

运行 *sysprep* 报错

执行 *sysprep.exe* 时，报错，无法继续下去。网上说是要把服务 *windows media player sharing* 禁用掉，但我禁用后仍然无效，据说 *sysprep* 执行是有次数限制，不知是否为这个限制所致。

`注意` ： WIN10使用此法无效，亲测