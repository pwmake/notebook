---
layout: post
title: Nginx搭建webdav服务
tags: [Skill]
date: 2017-11-07 11:24:35
feature-img: "https://images.pexels.com/photos/167259/pexels-photo-167259.jpeg?cs=srgb&dl=blur-blurry-close-up-167259.jpg&fm=jpg"
---

鉴于在不同电脑间经常会下载 *PDF* 电子书，但苦于无法统一管理。遂打算建立一个自用网盘，眼看一系列的搭建教程，庞大而复杂，部分还得依赖客户端，令人倍感沮丧。偶然间得知 `webdav` ，[专业概念](https://zh.wikipedia.org/wiki/WebDAV) 略过不讲，人话解释是 `远程网络硬盘` 。

<!--more-->

## 安装与配置

系统环境： *ubuntu 16.04* 

安装 *nginx* 

```shell
# 务必安装完整版
$ sudo apt install nginx-full
```

开启 `gzip` 

```shell
$ sudo sed -i '/gzip_/ s/#\ //g' /etc/nginx/nginx.conf
```

创建 `webdav` 虚拟主机

```shell
$ sudo cat > /etc/nginx/conf.d/webdav.conf < EOF
server {
    listen 80;
    listen [::]:80;
	  autoindex on;
	  charset utf-8;

    server_name dav.your_domain;

	# 账号认证
    auth_basic              realm_name;
    auth_basic_user_file    /etc/nginx/.passwords.list;

    dav_methods     PUT DELETE MKCOL COPY MOVE;
    dav_ext_methods PROPFIND OPTIONS;
    dav_access      user:rw group:rw all:r;

	# 临时中转目录，提高安全性；亦可取消
    client_body_temp_path   /tmp/nginx/client-bodies;
    
    # 上传文件的最大容量限制，0为不限制
    client_max_body_size    0;
    
    create_full_put_path    on;

	# 注意建立目录并给 www-data 权限
    root /data/webdav; 
}
```

创建账号认证文件

```shell
# 创建新用户
$ echo -n 'username:' | sudo tee /etc/nginx/.passwords.list

# 设定用户密码
$ openssl passwd -apr1 | sudo tee -a /etc/nginx/.passwords.list
Password: # 输入用户的密码
```

重启 *nginx* 即可。

## MacOS使用

在 **Finder** 中，选取“前往”>“连接服务器”，在“服务器地址”栏中输入服务器的地址，然后点按“连接”。

## Ubuntu使用

`方法一：nautilus` 

在 **nautilus** 资源管理器中，选取“文件”>“连接服务器”，在“服务器地址”栏中输入服务器的地址，然后点按“连接”。

**缺点** ：无法显示中文字符。

`方法二：davfs2`

安装

```shell
$ sudo apt-get install davfs2
```

挂载

```shell
$ sudo mount -t davfs -o uid= http://example/dav /mnt/webdav
```

-o选项设置挂载目录的拥有者为，使得非root用户拥有可写权限。

建议对于davfs2，禁用锁操作。需要编辑/etc/davfs2/davfs2.conf

```shell
use_locks       0
```

## Windows使用

1. 修改注册表，更改认证等级，粘贴以下文件，另存为 `webdav.reg` ，再执行之。

```powershell
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WebClient\Parameters]

"UseBasicAuth"=dword:00000001
"BasicAuthLevel"=dword:00000002
```

2. 进入`控制面板 -> 系统和安全(或系统和维护) -> 管理工具 -> 服务`，打开的弹窗中，找到 `WebClient`，启动之(如果已停止的话)，并且最好设置`启动类型`为`自动`。
3. 进入`我的电脑 (计算机)`，找到并点击 `映射网络驱动器` ，选择 `连接到可用于存储文档和图片的网站` ，按提示操作即可。

## 尾巴

如果你理解局域网共享 `smb` 之类的，则对于 `webdav` 同样能理解，局域网管理与传输文件，比 *FTP*  好用且高效。

> 如果你处于一个有多台电脑的办公室内，或者在家中也有多台电脑，那么总不可避免互相传输文件，最简单的方法是通过 QQ 等工具，但如果需求稍高，比如想像自己的文件夹一样，通过 WebDAV 来实现就显得非常简单了。

如果搭建在自己的 *VPS*  ，则可以作为远程网盘使用咯。单纯浏览 or 下载文件，则直接在浏览器打开地址即可。

如果觉得太复杂，可使用 **Parkomat** 快速搭建。

### 参考链接

> [davfs2 开机自动挂载](http://ajclarkson.co.uk/blog/auto-mount-webdav-raspberry-pi/)
>
> [win系统挂载 webdav](https://faq.bitcron.com/read/webdav)
>
> [Parkomat快速搭建WebDav服务](https://www.appinn.com/parkomat/)