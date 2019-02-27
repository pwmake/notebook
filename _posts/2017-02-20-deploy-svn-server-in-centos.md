---
title: 部署SVN Server
tags: [Linux]
date: 2017-02-20 22:05:14
layout: post
feature-img: "assets/img/pexels/startrails.jpg"
---

> 旧版此文是基于 *CentOS* 的，后转为 *Ubuntu* 

聊聊SVN服务端的部署...

<!--more-->

## Apache

必须安装 *Apache2* ，原有 *Nginx* 做主要的 *web* 服务时，可借助反向代理来解决。

```shell
$ sudo  apt install apache2 subversion libapache2-svn
```

记得添加一行记录，否则启动时 *Apache2* 会报错。

```shell
$ sudo echo 'ServerName 127.0.0.1' >> /etc/apache2/apache2.conf
```

创建虚拟主机，`/etc/apache2/sites-available/svn.conf`

```shell
# /etc/apache2/sites-available/svn.conf
listen 127.0.0.1:8888  # 设置监听的端口
<Location /svn>
  DAV svn
  SVNParentPath /data/svn
  AuthType Basic
  AuthName "Welcome to Svn Repository"
  AuthzSVNAccessFile /data/svn/authz
  AuthUserFile /data/svn/passwd
  Require valid-user
</Location>
```

启用该虚拟主机配置，并重启服务

```shell
$ sudo a2ensite svn.conf

$ sudo systemctl restart apache2
```

## SVN

设置通用的配置，如 `用户信息` ， `权限信息` ；继续按照 *Apache* 中的 目录设定来。

创建 *SVN*  主目录

```shell
# 创建主目录
$ mkdir /data/svn
```

创建一个示例仓库

```shell
$ cd /data/svn

$ svnadmin create repo1

$ chown -R www-data:www-data repo1
```

创建 `用户信息` 文件

```shell
# 创建示例用户 user001   -c 指定创建文件
$ htpasswd -cm /data/svn/passwd  user001 

$ chown root:www-data /data/svn/passwd

$ chmod 640 /data/svn/passwd

```

如果想要添加新的用户 或者 修改现有用户 ，则执行

```shell
$ htpasswd -m /data/svn/passwd user002
```

设置 `权限信息`，从 示例仓库 中拷贝一个配置文件模板，并做如下设置

```shell
$ sudo cp /data/svn/repo1/conf/authz /data/svn/authz

# 权限设置
$ vim /data/svn/authz
[groups]
admin = jeff
reader = reader

[/]
@admin=rw

# 指定某个仓库的权限
[repo1:/]
@server=rw
@reader=r
```

在 `/data/svn` 目录下，可对所有仓库、所有用户做 `全局设定`，亦可在每个仓库下的配置目录中单独设定。


## Nginx

借助 *nginx* 来实现反向代理，或添加 *SSL* 之类的。虚拟主机配置如下：

```shell
server {
   listen 80;
   server_name your_svn_server_name;
   location / {
     proxy_pass http://127.0.0.1:8888;
     proxy_set_header Host $host;
   }
}
```

SVN 自带了一个服务：`svnserve`，用起来略蛋疼，开启如下：

```shell
$ killall svnserver && svnserve -d -r /data/svn

# 指定SVN主目录
$ svnserve -d -r /data/svn 

# 防火墙开启指定端口
$ sudo ufw allow 3690
```

## 小结

直接换系统来部署，感觉使用上比 *CentOS* 会更好，有查找过相关的 网页前端 来管理 *SVN* 服务端，均不理想，很多都是不再维护已有两三年了。