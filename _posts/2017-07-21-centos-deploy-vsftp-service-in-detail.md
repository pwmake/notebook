---
layout: post
title: centos vsftp 配置
date: 2017-07-21 12:51:14.075000
tags: [Linux]
feature-img: "assets/img/pexels/startrails.jpg"
---


搭建ftp主要有两种情况

1. 使用操作系统内的用户「操作简便」
2. 使用虚拟用户「安全性高」

<!--more-->

分别就两种情况进行讨论，实践环境为

- Centos 6.8 64位
- yum 安装 vsftpd



## 本地用户

- 安装vsftpd

```bash
$ yum -y install vsftpd
```

- 开启必要的配置

```shell
# /etc/vsftpd/vsftpd.conf 开启以下配置
anonymous_enable=NO
local_enable=YES
write_enable=YES
chroot_local_user=YES
```

`敲黑板的重点` ： 务必注意，配置文件中，每条记录，行末，等于两边不能有多余的空格，甚至是注释行末尾也不能有多余的空格，否则报错。隐性大坑啊。

- 添加用户

```Shell
# 创建ftp用户的家目录
$ mkdir /data/ftp

# 添加ftp用户
$ useradd -d /data/ftp/ftp_user ftp_user
$ passwd ftp_user
```

- 重启服务

```shell
$ /etc/init.d/vsftpd restart
$ chkconfig --levels 2345 on
```

- 检查selinux状态

```shell
# 关闭selinux 
$ sed -i '/SELINUX/ s/enforing/disabled/g' /etc/sysconfig/selinux

# 不重启的情况下禁用
$  setenforce 0 
```

- 连接测试

使用 ftp 命令或者 ftp客户端来测试吧。注意如果开启了防火墙的话，放行端口 21 噢。

小结：此种临时适合内部网络临时开启使用一下，如果是部署在公网的话，则必要使用以下的虚拟用户模式了。公网讲求的是安全性。



## 虚拟用户

此模式下的步骤甚多，在此仅以最简洁的语言来描述，以便快速理解。

首先要理解两个概念： 宿主用户 与 虚拟用户，借用虚拟机的例子则更容易理解，宿主用户 相当于 *win10* ，虚拟用户相当于 *virtualbox* 中安装的各种虚拟机，你可以安装 *ubuntu* 、*win7* 等各种系统。ftp服务，对外，用户以虚拟用户的身份与之通信；对内，则以系统内的用户「即宿主用户」与之通信，形成了一个屏障式的隔离，N个虚拟用户对应的永远是同一个宿主用户，N个虚拟机对应的始终是同一个宿主机。

好了，话不多说，开始动手吧。

- 设置虚拟用户

借助 db4 来生成虚拟用户数据库，因此先安装之

```Shell
$ yum -y install db4 db4-utils
```

虚拟用户配置文件

```shell
$ touch /etc/vsftpd/vuser_passwd.txt
user1
passwd1

# 奇数行为用户名，偶数行为密码
```

生成数据库文件

```Shell
$ db_load -T -t hash -f /etc/vsftpd/vuser_passwd.txt /etc/vsftpd/vuser_passwd.db 
# 增删改虚拟用户 vuser_passwd.txt 再执行此命令即可
```

添加pam认证文件

```Shell
$ cp /etc/pam.d/vsftpd /etc/pam.d/vsftpd.bak

$ cat > /etc/pam.d/vsftpd << EOF
session    required     pam_loginuid.so
auth       required     /lib64/security/pam_userdb.so db=/etc/vsftpd/vuser_passwd
account    required     /lib64/security/pam_userdb.so   db=/etc/vsftpd/vuser_passwd
EOF
```

注意64位系统，需要写全路径，否则无法认证通过。

- 添加宿主用户

```Shell
# 设定ftp用户家目录
$ mkdir -pv /data/web

# 添加ftp用户，此处为ftpuser，注意不能让该用户登录
$ useradd -d /data/web -s /sbin/nologin ftpuser
```

- Vsftpd.conf 配置

备份原有的配置

```shell
$ cp /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf.bak
```

直接贴配置出来吧

```Shell
# /etc/vsftpd/vsftpd.conf 

# 禁止匿名访问
anonymous_enable=NO

# 本地用户可以访问，针对虚拟宿主用户
local_enable=YES

# 可写操作
write_enable=YES

# 上传后文件的权限掩码
local_umask=022

# 禁止匿名用户上传
anon_upload_enable=NO

# 禁止匿名用户建立目录
anon_mkdir_write_enable=NO

# 开启目录标语
dirmessage_enable=YES

# 开启日志记录
xferlog_enable=YES

# 使用端口20进行数据连接
connect_from_port_20=YES

# 禁止上传文件更改宿主
chown_uploads=NO

# 服务日志保存路径；需要手动建立并分配权限，否则启动失败
xferlog_file=/var/log/vsftpd.log

# 使用标准的日志记录格式
xferlog_std_format=YES

# vsftpd服务的宿主用户为手动建立的vsftpd用户，更改后权限也要改
nopriv_user=ftpuser

# 开启异步传输
async_abor_enable=YES

# 开启ASCII模式的上传和下载
ascii_upload_enable=YES
ascii_download_enable=YES

# 登录后的标语
ftpd_banner=come on and fuck me

# 禁止本地用户跳出ftp主目录
chroot_local_user=YES

# PAM服务的验证配置文件名，对应/etc/pam.d/vsftpd
pam_service_name=vsftpd

# 守护进程形式
listen=YES

# 启用被动模式
pasv_enable=YES
pasv_min_port=10000
pasv_max_port=10010
local_max_rate=200000

# 开启虚拟用户
guest_enable=YES

# 设定宿主用户
guest_username=ftpuser

# 虚拟用户的权限符合宿主用户
virtual_use_local_privs=YES

# 虚拟用户个人的vsftpd配置文件路径
user_config_dir=/etc/vsftpd/vuser_conf

# 启用黑白名单
tcp_wrappers=YES
```

授权给 ftp 家目录

```Shell
$ mkdir -pv /data/web && chown ftpuser:ftpuser /data/web
```

创建日志记录文件

```shell
$ touch /var/log/vsftpd.log && chown ftpuser:ftpuser /var/log/vsftpd.log
```

启用被动模式的话，则要在防火墙中添加该模式的端口放行，即

```shell
-A INPUT -m state --state NEW -m tcp -p tcp --dport 10000:10010 -j ACCEPT
```

- 虚拟用户配置

设定虚拟用户配置目录

```Shell
$ $ mkdir -pv /etc/vsftpd/vuser_conf
```

创建具体虚拟用户配置，此处假定为 「test」 用户

```shell
$ cat > /etc/vsftpd/vuser_conf/test << EOF
local_root=/data/ftp/test
write_enable=YES # 上传权限
local_umask=022
anon_world_readable_only=NO
anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_other_write_enable=YES
anonymous_enable=NO
download_enable=YES # 下载权限
EOF
```

可单独配置成仅上传 或者 仅下载

- 开启SSL

在 ubuntu 中使用 FTP图形化客户端时，默认强制要求连接 FTP 服务需要使用 SSL 。因此，在服务端配置`/etc/vsftpd/vsftpd.conf` 增加

```shell
ssl_enable=YES
rsa_cert_file=/etc/vsftpd/vsftpd.pem
ssl_ciphers=HIGH
require_ssl_reuse=NO
```

手动创建钥匙

```shell
$ cd /etc/vsftpd && openssl req -new -x509 -nodes -out vsftpd.pem -keyout vsftpd.pem -days 1095
```

- 权限处理

虚拟用户上传文件后，默认的权限值为644，没有执行权限；欲让其执行，则

在配置 `/etc/vsftpd/vsftpd.conf` 中增加

```shell
# 权限775 
local_umask=0002
file_open_mode=0777
```

- 群组处理

倘若某个虚拟用户目录是用以做 「web」开发的，则 「ftp」主目录的属主、群组修改为

```shell
$ chown ftpuser:www -R /data/web
```

虚拟用户上传的文件，让其具备与主目录相同的属主与群组设定，则只需要借助 SGID 来实现

```shell
$ chmod g+s /data/web
```

执行该命令后，则上传的文件、目录默认会继承该 属主 和群组。