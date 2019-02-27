---
layout: post
title: MacOS Sierra 安装apache+mysql+php
date: 2017-01-19 11:28:11
tags: [MacOS]
feature-img: "https://images.pexels.com/photos/265101/pexels-photo-265101.jpeg?cs=srgb&dl=art-business-cactus-265101.jpg&fm=jpg"
---

最近更新到 *sierra* 系统版本后，我发现有点不对劲，之前设定的 *apache* 用户主页打不了，怎么破？再者，之前安装的 *mysql* 无法启动，密码也忘记了，额……好杯具，好吧……只能重新部署下环境了，方便将来的使用。

**MacOS** 默认自带了 *apache* 和 *php* ，前者特别难用，本文只介绍通过`brew` 安装*apache* 、*mysql* 和 *php* 。

<!--more-->

## Homebrew

首先安装 `homebrew` 。

```bash
$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

查看其版本

```shell
$ brew --version
Homebrew 1.1.7
Homebrew/homebrew-core (git revision d8c1; last commit 2017-01-18)
```

检测配置是否正常，并根据提示进行操作，解决某些冲突，或者是重新建立关联

```shell
$ brew doctor
```

添加第三方的软件源

```shell
$ brew tap homebrew/dupes
$ brew tap homebrew/versions
$ brew tap homebrew/php
$ brew tap homebrew/apache
```

更新软件源，与 `apt update` 类似

```shell
$ brew update
```

如果你的执行 `brew update` 之后，很久都没反应的话，请重新安装 *homebrew* 再试。

## 安装apache

先把 *MacOS* 自带的 *apache* 给禁用掉

```shell
$ sudo apachectl stop
$ sudo launchctl unload -w /System/Library/LaunchDaemons/org.apache.httpd.plist 2>/dev/null
```

安装

```shell
$ brew install httpd24 --with-privileged-ports --with-http2
```

执行后请等待一段时间，系统会自动下载所需要的包并完成编译安装，安装完毕后，会显示以下的版本信息

```shell
/usr/local/Cellar/httpd24/2.4.25: 214 files, 4.5M, built in 6 minutes 38 seconds
```

这里的版本号（2.4.25）很重要，把新安装的 *apache* 添加到开机启动时要用到

```shell
$ sudo cp -v /usr/local/Cellar/httpd24/2.4.25/homebrew.mxcl.httpd24.plist /Library/LaunchDaemons
$ sudo chown -v root:wheel /Library/LaunchDaemons/homebrew.mxcl.httpd24.plist
$ sudo chmod -v 644 /Library/LaunchDaemons/homebrew.mxcl.httpd24.plist
$ sudo launchctl load /Library/LaunchDaemons/homebrew.mxcl.httpd24.plist
```

好了，现在已经启动了 *apache* ，赶紧在浏览器中输入 `http://localhost` 试试。

### 调试与启动

查询进程是否已经启动

```shell
$ ps -aef | grep httpd
```

强制重启

```shell
$ sudo apachectl -k restart
```

查看错误日志

```shell
$ tail -f /usr/local/var/log/apache2/error_log
```

如果仍然无法打开 *http://localhost* 时，请在配置文件中确认默认端口使用的是 *80*，配置文件为`/usr/local/etc/apache2/2.4/httpd.conf`。 

开启、重启与关闭命令

```shell
$ sudo apachectl start
$ sudo apachectl stop
$ sudo apachectl -k restart
```

### Apache配置

为日常使用方便，做以下设置

```shell
# 打开 /usr/local/etc/apache2/2.4/httpd.conf

# 默认的 DocumentRoot "/usr/local/var/www/htdocs"
# 改成家目录的Sites
DocumentRoot "/Users/your_user/Sites"
# 同时修改 Directory的目录名
<Directory /Users/your_user/Sites>
# 同时把该<Directory> 里头的 AllowOverride改成all
AllowOverride All

# 开启把重定向的模块
LoadModule rewrite_module libexec/mod_rewrite.so
```

### 用户与群组

修改 *apache* 配置，修改用户与群组

```shell
User your_user
Group staff
```

### 建立Sites目录

```shell
$ mkdir ~/Sites
$ echo "<h1>My User Web Root</h1>" > ~/Sites/index.html
$ sudo apachectl -k restart
```

赶紧打开浏览器测试下吧。

## 安装PHP

由于本人甚少使用PHP，如需要安装多版本的php，并使用版本切换脚本随时切换php版本的话，请参考末尾的原文链接。此处仅安装 *PHP 5.6* 稳定版。

执行安装，等待一段时间，系统会自动下载、编译安装

```shell
$ brew install php56 --with-httpd24
```

修改配置文件中的时区

```shell
$ vim /usr/local/etc/php/5.6/php.ini
date.timezone = Asia/Shanghai
```

修改*Apache* 配置， `/usr/local/etc/apache2/2.4/httpd.conf` 

```shell
# php安装后会自动添加模块
LoadModule php5_module /usr/local/Cellar/php56/5.6.29_5/libexec/apache2/libphp5.so

# 把上面的，修改成简短的形式
LoadModule php5_module /usr/local/opt/php56/libexec/apache2/libphp5.so

# 增加 index.php
<IfModule dir_module>
    DirectoryIndex index.php index.html
</IfModule>

# 紧接着上面 添加
<FilesMatch \.php$>
    SetHandler application/x-httpd-php
</FilesMatch>
```

重启apache，并访问测试

```shell
$ sudo apachectl restart
$ echo '<?php phpinfo();' > ~/Sites/info.php
```

## 安装Mysql

目前已经使用`MariaDB` 代替原来的 *MySQL* ，因此我们直接安装前者

```shell
$ brew install mariadb
# 初始化Mysql
$ mysql_install_db
```

启动

```shell
$ mysql.server start
Starting MySQL
. SUCCESS
```

强烈推荐使用[SequelPro](http://www.sequelpro.com/) ， 这货很强大，并且免费的，可视化操作MySQL数据库的软件。

接着是修改密码

```shell
$ /usr/local/bin/mysql_secure_installation
```

关闭MySQL

```shell
$ mysql.server stop
```

## Apache虚拟主机

在Apache配置中，开启以下两个选项

```shell
LoadModule vhost_alias_module libexec/mod_vhost_alias.so

# Virtual hosts
Include /usr/local/etc/apache2/2.4/extra/httpd-vhosts.conf
```

编辑虚拟主机配置文件，添加自己想要设定的主机名

```shell
$ vim /usr/local/etc/apache2/2.4/extra/httpd-vhosts.conf

<VirtualHost *:80>
    DocumentRoot "/Users/your_user/Sites"
    ServerName localhost
</VirtualHost>

<VirtualHost *:80>
    DocumentRoot "/Users/your_user/Sites/grav-admin"
    ServerName grav-admin.dev
</VirtualHost>
```

重启*Apache* 测试效果吧。

## 其他

关于 *Dnsmasq* 、*APC Cache* 、*YAML* 、*Xdebug* 的安装，这里就不弄了。

## 参考链接

> [Part 1: install apache php](https://getgrav.org/blog/macos-sierra-apache-multiple-php-versions)
>
> [Part 2: install mysql, dnsmasq, APC Cache, Yaml and more](https://getgrav.org/blog/macos-sierra-apache-mysql-vhost-apc)