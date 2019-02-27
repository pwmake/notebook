---
title: 科学上网之shadowsocks
layout: post
date: 2016-11-29 00:00:00
tags: [Skill, Freedom]
feature-img: "https://images.pexels.com/photos/255514/pexels-photo-255514.jpeg?cs=srgb&dl=aeroplane-aircraft-airplane-255514.jpg&fm=jpg"
---


打开新世界的大门，`自由` 点击就送……嘿嘿……

<!--more-->

本文只讲部分要点，快速了解使用说明请点击：[SS使用说明](https://github.com/shadowsocks/shadowsocks/wiki/Shadowsocks-%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E) ； 此外一定要阅读 [这个](https://wiki.archlinux.org/index.php/Shadowsocks_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)) ！！！

## 服务端

执行 `pip install shadowsocks` 安装即可，主要是 **内核优化** 与  **BBR算法**。

### 内核优化

修改 `/etc/sysctl.conf` 

```shell
sudo cat >> /etc/sysctl.conf << EOF
# max open files
fs.file-max = 51200
# max read buffer
net.core.rmem_max = 67108864
# max write buffer
net.core.wmem_max = 67108864
# default read buffer
net.core.rmem_default = 65536
# default write buffer
net.core.wmem_default = 65536
# max processor input queue
net.core.netdev_max_backlog = 4096
# max backlog
net.core.somaxconn = 4096
# resist SYN flood attacks
net.ipv4.tcp_syncookies = 1
# reuse timewait sockets when safe
net.ipv4.tcp_tw_reuse = 1
# turn off fast timewait sockets recycling
net.ipv4.tcp_tw_recycle = 0
# short FIN timeout
net.ipv4.tcp_fin_timeout = 30
# short keepalive time
net.ipv4.tcp_keepalive_time = 1200
# outbound port range
net.ipv4.ip_local_port_range = 10000 65000
# max SYN backlog
net.ipv4.tcp_max_syn_backlog = 4096
# max timewait sockets held by system simultaneously
net.ipv4.tcp_max_tw_buckets = 5000
# turn on TCP Fast Open on both client and server side
net.ipv4.tcp_fastopen = 3
# TCP receive buffer
net.ipv4.tcp_rmem = 4096 87380 67108864
# TCP write buffer
net.ipv4.tcp_wmem = 4096 65536 67108864
# turn on path MTU discovery
net.ipv4.tcp_mtu_probing = 1
# for high-latency network 
net.ipv4.tcp_congestion_control = hybla
# for low-latency network, use cubic instead
#net.ipv4.tcp_congestion_control = cubic
EOF

# 令参数生效
sudo sysctl -p
```

增加文件大小限制

```shell
cat >> /etc/security/limits.conf << EOF
* soft nofile 51200
* hard nofile 51200
EOF

ulimit -n 51200
```


### 开启BBR算法

升级 Linux 内核为至少 4.9 版本，以便开启谷歌的 BBR拥塞算法，加快网络的处理速度。

```shell
## 查询内核版本号
$ sudo apt search linux-image-4.10 | grep generic
## 选择一个版本号安装,例如
$ sudo apt install linux-image-4.10.0-42-generic

# 查询当前内核版本
$ uname -r
4.4.0-57-generic

# 删除当前使用的内核
$ apt-get remove linux-image-4.4.0-57-generic

# 更新grub系统引导文件并重启
$ update-grub
$ reboot 

# 开启bbr
$ echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
$ echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf

$ sysctl -p
$ sysctl net.ipv4.tcp_available_congestion_control 
$ lsmod | grep bbr # 查询到有bbr就可以了
```

推荐使用谷歌的BBR算法，取代 *hybla* 。


## 客户端

各平台的应用一览：

1. [Windows](https://github.com/shadowsocks/shadowsocks-windows/releases) 
2. [MacOS](https://github.com/shadowsocks/ShadowsocksX-NG/releases)
3. [Android](https://github.com/shadowsocks/shadowsocks-android/releases)
4. [浏览器](https://github.com/FelisCatus/SwitchyOmega) 与 [gfwlist](https://github.com/gfwlist/gfwlist)
5. IOS：shadowrocket 咯

重点说说 Linux 下的客户端部署，首选 `systemd` 的方式。

守护进程脚本如下：`/lib/systemd/system/shadowsocks@.service`

```shell
[Unit]
Description=Shadowsocks Client Service
After=network.target

[Service]
Type=simple
User=nobody
ExecStart=/usr/bin/sslocal -c /etc/shadowsocks/%i.json
Restart=always

[Install]
WantedBy=multi-user.target
```

假设 SS 配置文件是: `/etc/shadowsocks/jeff.json` ，则执行

```shell
# 开启服务
$ sudo systemctl start shadowsocks@jeff

# 开机启动
$ sudo systemctl enable shadowsocks@jeff
```

除此之外，务必安装 `proxychains` 提高在命令行中翻墙体验。


## 小结

在路由器挂着ss，几个月下来，最直观的感受是，国外域名解析上很蛋疼，导致网站访问有点被拖慢了，哎……暂时找不到好的国外DNS，解析国外的网站。不通过路由器时，速度却又出乎意料地飞快。哎……路由器再怎么着也还是路由器……


最后一句话：自由的感觉真好，设置浏览器主页为 **chrome** 的感觉真是妙哉！