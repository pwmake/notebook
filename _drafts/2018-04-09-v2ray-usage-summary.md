---
layout: post
title: v2ray使用小结
date: 2018-04-09
tags: [HowTo]
feature-img: "https://images.pexels.com/photos/255514/pexels-photo-255514.jpeg?cs=srgb&dl=aeroplane-aircraft-airplane-255514.jpg&fm=jpg"
---

用了两个晚上的时间把 v2ray 阅读完毕，其平台式、模块化的组合，易用性与安全性极好，嗯嗯，果断得跟进使用啊。

<!--more-->

## 关于文档

先阅读 [白话文教程](https://toutyrater.github.io/) ，再阅读 [官方文档](https://www.v2ray.com/) 。前者通俗易懂，按图索骥，则可以完成 `V2RAY` 的搭建与日常使用。

接着再来一段 [长城攻防战](https://steemit.com/cn/@v2ray/6knmmb) 的历史以助兴，增长见识。 

看到官博写到 `v2ray` 升级为 `Project V`时，不禁会心一笑，哈哈，影史上的V先生?。

官方文档上推荐了一个优秀的 [v2ray配置生成器](https://htfy96.github.io/v2ray-config-gen/) ，文档阅读完毕后，真正建立自身的配置时，可使用之。

> 该生成器有部分参数未提及，需要手动添加。

## 关于安装

网上的一键安装脚本，可使用 [233blog v2ray安装与管理脚本](https://233blog.com/post/16/) ，但我本人没使用之。

官方推荐的脚本安装方法，仅支持 yum 或 apt 的 *linux* 系统，另外最好是支持 `systemd` 的，详情可参考 [安装v2ray](https://toutyrater.github.io/prep/install.html) 。

```bash
# 下载脚本
$ wget https://install.direct/go.sh

# 运行安装
$ sudo bash go.sh

# 版本升级时，则再跑一次即可
$ sudo bash go.sh
```

使用 Docker 安装会更简单方便，参考 [docker部署V2ray](https://toutyrater.github.io/app/docker-deploy-v2ray.html) ，一步步照着来即可。


## 关于原理

要理解两个东西：

1. 无论是SS 或 V2ray，需要借助规则列表来处理是否需要代理介入，V2ray自带路由 ，亦可自身实现。可阅读 [路由功能](https://toutyrater.github.io/basic/routing/) 。
2. 使用 `v2ray` 时，整个运行过程是：`浏览器打包 --(socks)--> V2Ray 客户端接收 -> V2Ray 客户端发出 --(VMess)--> V2Ray 服务器接收 -> V2Ray 服务器发出 --(Freedom)--> 目标网站`，可阅读 [VMess协议](https://toutyrater.github.io/basic/vmess.html)，这能帮助理解为啥区分 `inbound` 与 `outbound`。


## 关于配置

搞清楚什么是 `json` ，运行 **v2ray** 服务前可使用以下命令来检查 json 配置文件的正确性。
```bash
$ /usr/bin/v2ray/v2ray -test -config /etc/v2ray/config.json
V2Ray v3.16 (die Commanderin) 20180405
A unified platform for anti-censorship.
Configuration OK.
```

其次是 `UUID` 的生成，不能随便自个造，可使用在线版 [online uuid generator](https://www.uuidgenerator.net/) ，或者是 linux 命令版

```bash
$ cat /proc/sys/kernel/random/uuid
```

使用 `Vmess协议` 做最简化的配置，装好即可用，搭配客户端规则列表即可上手使用，参考 [这个](https://toutyrater.github.io/basic/vmess.html) 。

使用 `Shaodowsocks协议`，也很简单，感觉比原本 **shadowsocks** 安装来得更简化，参考 [这里](https://toutyrater.github.io/basic/Shadowsocks.html)，需要自行开启 `BBR` 算法噢。另外要特别注意算法上的变更，使用`AEAD`。

> 因为协议漏洞，Shadowsocks 已放弃 OTA 转而使用 AEAD,但 V2Ray 依然兼容 OTA 和 AEAD，建议使用 AEAD (method 为 aes-256-gcm、aes-128-gcm、chacha20-poly1305 即可开启 AEAD), 使用 AEAD 时 OTA 会失效。

使用目前主流较安全的方式：`WebSocket+TLS+Web`，参考 [这里](https://toutyrater.github.io/advanced/wss_and_web.html) ，需要 `SSL`、`Nginx` 相关知识。

还有个 CDN 方式的，俺不会搞，不表之。选择哪种方式，自己决定吧，或者去参考网友的；可以多种协议组合一起使用。

## 关于客户端

官方文档有 [介绍](https://www.v2ray.com/ui_client/) ，**Android** 的支持较为简单，可以分应用，或者绕开大陆地址。

## 关于路由器

有大神已做出梅林的插件，需要开启 [虚拟内存](http://koolshare.cn/thread-42723-1-1.html) 辅助使用！！

`LEDE` 中使用时，如果配置加了伪装，则需要开启 *v2ray* 的 `任意门` 模式，详情参考 [这里](http://www.right.com.cn/forum/forum.php?mod=viewthread&tid=310440&extra=page%3D1&page=1)。 

## 小结

万事开头难，过程与结果更加难；这类东东，实践出真知啊！

#### 更新记录
> 2018-05-01
> 增加路由器使用 V2ray 的描述
>
> 2018-06-24
> 增加LEDE使用 v2ray 的說明
