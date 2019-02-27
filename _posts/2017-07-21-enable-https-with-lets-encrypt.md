---
layout: post
title: 使用Let's Encrypt开启HTTPS
date: 2017-07-21 23:12:55.533000
tags: [Skill]
feature-img: "https://images.pexels.com/photos/4291/door-green-closed-lock.jpg?cs=srgb&dl=closed-coming-soon-door-4291.jpg&fm=jpg"
---


Let's Encrypt 是一个于2015年三季度推出的数字证书认证机构，将通过旨在消除当前手动创建和安装证书的复杂过程的自动化流程，为安全网站提供免费的SSL/TLS证书。

申请 Let's Encrypt 证书非常简单，并且是免费的，官方提供了自动化的工具进行配置。每次生成的证书只有90天的有效期，但可以通过设定计划任务自动续期，完全是一劳永逸的操作呢。

<!--more-->

在操作之前，先得理清几个问题：

## 什么是HTTP 和 HTTPS

HTTP 是一个网络协议，用于传输 web 内容，例如文字、图片等等信息。HTTP使用的是明文传输，因而存在的两个弊端，能够被人窃听「准确地说是嗅探」到，从而窃取用户的信息，如登录时的用户名和密码等等；或者是篡改信息。而HTTPS则是在HTTP的基础上，增加了 SSL「Secure Sockets layer」/TSL「Transport Layer Security」，解决明文传输的弊端，增强安全性。

## 什么是数字证书

数字证书「Public Key Certificate」 ，又叫身份证书、公钥证书等等，详情可查阅维基百科，用人话来讲相当于是一张身份证，证明你是谁。而在网络上则用来证明你是安全的，可靠的网站。那证书由谁颁发呢？由权威的`证书机构` 「Certificate Authority」颁发的。证书与证书之间存在信任关系，道理类似老板信任经理，经理信任主管，则老板也信任主管。Let's Encrypt即是一个证书机构。那数字证书有什么用呢？它是用来验证开启了HTTPS的网站是否可信，假如有个钓鱼的银行网站，域名与真的完全一致，同样是HTTPS的，但如果访问它时，浏览器提示该证书不可信，则证明该网站是个假网站。

## 为什么要使用HTTPS

主要是为了防止`嗅探` 和 `篡改` 。嗅探是防止别人窃取你的用户信息。 篡改是为了防止运营商劫持的流氓行为，影响网站访问者的用户体验。开启HTTPS会不会影响网站的性能呢？正所谓“存在即合理”，HTTPS如此盛行，证明有方法能够解决性能问题，本人了解有限，就不对此进行展开了。

前面讲这么多废话是为啥？我们需要向 Let's Encrypt证书机构申请一张免费的数字证书，把证书信息添加到网站的HTTPS里头去，当网站访问者浏览网页时，在浏览器地址栏会显示一个绿色的小锁。正因为有免费证书的存在，才让我有动力为网站开启HTTPS，否则……说白了，还是因为穷。毕竟是个人博客网站而已。

## 生成Let's Encrpyt证书

官方网站上有简要的[使用指南](https://letsencrypt.org/getting-started/)， 根据是否有`shell` 权限进行区分为两种方式。我选择的是有`shell` 权限的，即直接连接到服务器终端上。操作系统使用的是 `ubuntu 16.04` ，web服务使用的是 `apache` 。在`certbot`[自动化工具](https://certbot.eff.org/#ubuntuxenial-apache)中选择相应的选项即可。

### 安装

```bash
$ sudo apt-get update
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository ppa:certbot/certbot
$ sudo apt-get update
$ sudo apt-get install python-certbot-apache 
```

### 配置

```Shell
# 自动化配置，按提示操作
$ sudo certbot --apache

# 如果想手动配置，则执行
$ sudo certbot --apache certonly
```

配置成功生成后，到`apache` 的配置目录下，即可看到对应网站配置多了一个SSL的文件，形如

```Shell
tt.conf  
tt-le-ssl.conf
```

### 自动续期

先进行续期测试

```shell
$ sudo certbot renew --dry-run
```

如果没问题了，再把下面的命令添加到计划任务中去

```Shell
$ sudo certbot renew
```

`crontab` 计划任务

```shell
# 假设生成配置的当天时间是 2017-07-07 14:00:00
0 14 7 */3 * /usr/bin/certbot renew
```

### django网站配置报错

如果apache配置的是 django 网站，则生成证书的过程当中会报错

```Shell
Letsencrypt cannot deal with apache's WSGI configuration
```

它无法处理 WSGI 的配置，假设django网站的apace配置文件是 `tt.conf` ，解决方法是

- 生成配置前

把以下三行从 `tt.conf` 中移除

```Shell
 WSGIScriptAlias / /your_direcotry/wsgi.py
 WSGIDaemonProcess your_configuration
 WSGIProcessGroup your_project_name
```

- 生成配置后

把上述三行添加回 `tt.conf` ，同时把以下两行添加到 `tt-le-ssl.conf` 中

```shell
WSGIDaemonProcess your_configuration
WSGIProcessGroup your_project_name
```

## 参考链接

> [维基百科：Let's Encrypt](https://zh.wikipedia.org/wiki/Let%27s_Encrypt)
>
> [维基百科：数字证书](https://zh.wikipedia.org/wiki/%E5%85%AC%E9%96%8B%E9%87%91%E9%91%B0%E8%AA%8D%E8%AD%89)
>
> [Let's Encrpyt官方网站](https://letsencrypt.org/)
>
> [编程随想：扫盲HTTPS 和 SSL/TLS协议](https://program-think.blogspot.com/2014/11/https-ssl-tls-1.html)
>
> [编程随想：数字证书及CA的扫盲介绍](https://program-think.blogspot.com/2010/02/introduce-digital-certificate-and-ca.html)