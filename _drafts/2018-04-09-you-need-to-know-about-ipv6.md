---
layout: post
title: IPV6使用指南
date: 2018-03-13
tags: [Linux]
feature-img: "assets/img/pexels/startrails.jpg"
---


应公司业务需要「苹果 IOS 应用程序支持 IPv6 DNS64/NAT64 网络」，一番折腾后，整理使用文档如下，以记之。


<!--more-->


## 常见问题

- 什么是 *IPV6* ??

详见维基百科：[IPv6](https://zh.wikipedia.org/wiki/IPv6) 。简述是地址容量远比 IPv4 大得多了。

- 如何查询 IPv6 地址？

Linux 的，执行 `ifconfig` 即可

```bash
$ ifconfig
ens4      Link encap:Ethernet  HWaddr 42:01:0a:8c:00:03  
          inet addr:10.140.0.3  Bcast:10.140.0.3  Mask:255.255.255.255
          inet6 addr: fe80::4001:aff:fe8c:3/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1460  Metric:1
          RX packets:777217 errors:0 dropped:0 overruns:0 frame:0
          TX packets:627225 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:1209781021 (1.2 GB)  TX bytes:1153915212 (1.1 GB)
```

如果没看到

```bash
inet6 addr: fe80::4001:aff:fe8c:3/64 Scope:Link
```

证明不支持 IPv6。

- 如何区分IPv6地址是否为公网地址？

形如 `fe80::xxxxx:xx:xxx` ，均为内网地址。

而形如 `2001::xxxx:xx:xxx` 的则为公网地址。

可通过执行命令查看，Scope 代表范围。

```bash
$ ifconfig | grep inet6
    inet6 addr: fe80::f816:3eff:fe0d:6619/64 Scope:Link  # 内网
    inet6 addr: 2001:470:18:b83::2/64 Scope:Global  # 公网
```

- 如何 ping IPv6地址？

当目标地址为内网时，则　`ping6 -I <网卡名称> 内网IPv6地址` 。

当目标地址为公网时，则　`ping6 ipv6.google.com` 即可。

部分 Linux 发行版可能没有 `ping6` 命令「如 arch」，则执行 `ping -6` 。

- IPv6 的DNS

使用谷歌的  [公共DNS](https://developers.google.com/speed/public-dns/docs/using)

2001:4860:4860::8888
2001:4860:4860::8844

## 如何使用

摘抄自 [如何配置以实现纯 IPv6-Only 网络访问](https://guozeyu.com/2016/08/talk-about-config-ipv6-on-server/) 中的IPv6描述：

> 严格上来讲，IPv6-Only 网络下只能连接上 IPv6 地址，这也就意味着 DNS 缓存服务器也必须是 IPv6 地址，只能连接上支持 IPv6 的服务器。如果要解析一个域名，这个域名本身及其所属的根域名的 DNS 服务器也必须统统支持 IPv6。总之，在整个过程中不存在 IPv4。所以想要支持 IPv6-Only 网络，几乎需要在所有的环节下功夫。

检测域名所使用的 DNS 服务器是否支持 IPv6， 如本站的

```bash
$ dig -t AAAA `dig zshmobi.com ns +short` +short
2620:74:19::33
2001:502:cbe4::33
```

无任何内容则代表不支持 IPv6。

## 方式一：直接配置 IPv6

让网站、API服务支持 IPv6，可以通过配置 IPv6 来实现，鉴于天朝实情，无法从服务器那直接分配公网 IPv6 地址，只能使用 **[Tunnel Broker](https://www.tunnelbroker.net/)**  来实现。

- 开启 IPv6

配置之前，先确保操作系统支持 IPv6，可参考 [CentOS开启 IPv6支持](http://coolnull.com/4474.html) 。

```bash
# /etc/sysctl.conf 添加关闭禁用 ipv6
$ cat >> /etc/sysctl.conf << EOF
net.ipv6.conf.all.disable_ipv6 = 0
net.ipv6.conf.default.disable_ipv6 = 0
net.ipv6.conf.lo.disable_ipv6 = 0
EOF

# 添加支持ipv6
$ echo 'NETWORKING_IPV6=yes' >> /etc/sysconfig/network

# 查看效果
$ sysctl -p
error: "net.ipv6.conf.all.disable_ipv6" is an unknown key

# 如有上述报错，则修改内核模块，开启IPV6
# 加载部分内核
$ modprobe ipv6
$ modprobe nf_conntrack
$ modprobe nf_conntrack_ipv4

cat > /etc/modprobe.d/ipv6.conf << EOF
# Anaconda disabling ipv6
options ipv6 disable=0
EOF

# 检查内核模块是否支持 IPv6
$ lsmod | grep ipv6
ipv6 336282  49 sit

# 查询网卡信息是否有 inet6
```

- 获取 IPv6地址

*Tunnel Broker* 的申请与配置，可参考 [阿里云CentOS添加ipv6隧道](http://coolnull.com/4476.html) ，注意之处有：

**Debian/Ubuntu** 用户，配置信息可直接添加到网卡设置

```bash
# 追加 IPv6 的信息
$ cat > /etc/network/interfaces << EOF
auto he-ipv6
iface he-ipv6 inet6 v4tunnel
        address <Client IPv6 Address>
        netmask 64
        endpoint <Server IPv4 Address>
        local <Client IPv4 Address>
        ttl 255
        gateway <Server IPv6 Address>
EOF
```

**CentOS** 的用户，则在命令行执行即可；可写到脚本中，设置开机启动。

```bash
# 以 root 用户执行
$ modprobe ipv6
$ ip tunnel add he-ipv6 mode sit remote <Server IPv4 Address> local <Client IPv4 Address> ttl 255
$ ip link set he-ipv6 up
$ ip addr add <Client IPv6 Address> dev he-ipv6
$ ip route add ::/0 dev he-ipv6
$ ip -f inet6 addr
```

`针对云服` ：有外网地址 与 *VPC内网地址* 的情况，则 `<Client IPv4 Address>` 填写内网地址。阿里云如此设置有效，谷歌云无效，其他云服未知。

服务器添加 IPv6 的 DNS

```bash
$ cat > /etc/resolv.conf << EOF
nameserver 2001:4860:4860::8888
nameserver 2001:4860:4860::8844
EOF
```

现在可以尝试 *ping6 ipv6.baidu.com*  能否正常响应了。

- 添加域名AAAA解析

成功获取 ipv6 公网地址后，到所属于域名商，添加对应域名的 AAAA 解析记录，指定到上面获取的 ipv6地址。

- 设置web服务

检查 *nginx* 是否支持 IPv6，如果不支持，则需要重新编译安装之。

```bash
$ nginx -V
--with-ipv6 # 查看是否有此项
```

全局指定使用v4 与 v6

```bash
listen 80;
listen [::]:80;
```

指定某个虚拟主机使用 IPv6

```bash
# 填写指定的 IPv6
server {
    listen [2001:470:18:b88::2]:80 default_server ipv6only=on;
    .....
}
```

不同 nginx 版本有不同的设定，视情况而定。

- IPv6测试

访问 [ipv6-test](http://ipv6-test.com/validate.php) 测试网站是否设置正确，如下图所示

![测试结果](https://i.loli.net/2018/03/22/5ab30a8886bcc.png)

如果 `IPv6 DNS server` 一栏中显示 `X` ，则参考 [用HE提供的免费DNS解析服务通过IPv6 DNS检测](https://bbs.aliyun.com/read/313524.html?spm=a2c4e.11155515.0.0.8ol4k0) 。与上述 *Tunnel Broker* 为同一主站，注意之处有：

1. 创建 `new domain` 时只能添加一级域名，形如  `foo.com` 。
2. 添加  **AAAA Record** 时则指定二级域名 与 IPv6地址绑定。
3. 在域名服务商中 修改or增加 NameServer解析。 

关于第三点，部分域名服务商的设置可参考 [万网域名修改 DNS 方法](https://help.aliyun.com/knowledge_detail/39845.html?spm=a2c4e.11155515.0.0.9lE9JU) 。但某些服务商不支持此操作，怎么破？例如我司用的 `DNSPOD` 。

**DNSPOD** 默认使用的的 `nameserver` 是  `f1g1ns1.dnspod.net.` 但它只能解析 IPv4的，而现在需要借用 HE 提供的 `ns1.he.net` 来解析我们上述获取到的 IPv6 地址。So，添加如下几条 NS 记录：

```bash
主机记录	记录类型	记录值         TTL
@        NS     ns1.he.net.   600 
@        NS     ns2.he.net.   600 
@        NS     ns3.he.net.   600 
@        NS     ns4.he.net.   600 
@        NS     ns5.he.net.   600 
```

确保以上记录处于使用的状态噢。鉴于生效需要一定时间，设置完毕后，自行隔多长时间再测试下。

## 方法二：使用 CDN/HTTP Proxy

本人暂未使用此法，遂继续摘抄  [如何配置以实现纯 IPv6-Only 网络访问](https://guozeyu.com/2016/08/talk-about-config-ipv6-on-server/) 中的描述

>  使用免费的 CloudFlare，开启 CDN 功能，使用[CNAME/IP 接入](https://cf.tlo.xyz/)甚至还可以配置 IPv4 回源，仅使用 IPv6 CDN)，不过这样的话连 DNS 在内的所有服务都能支持 IPv6 了，类似的还有 Akamai，它们在代理了之后都能给你 IPv6 支持。

详细设置请参考 [接入教程](https://wiki.tloxygen.com/CloudFlare_%E6%8E%A5%E5%85%A5/%E6%95%99%E7%A8%8B) ， 至于为何要使用此法呢？

> 由于 App Store 要求应用必须支持纯 IPv6 网络，IPv6 已经成为刚需。然而 [Hurricane Electric Tunnel Broker](https://tunnelbroker.net/) 的速度在国内奇慢（国内连接的话香港节点由于绕道 500ms，最快的洛杉矶也 300ms），加上 TCP 的握手延迟实在是不堪设想，且还需要独立 IP 和服务端配置，实在是麻烦。使用本站的 CloudFlare 的 IP 接入可以轻松解决以上难题，且还会自动部署 AnyCast CDN，自动缓存静态内容，节省源站带宽。

## 小结

本文只是汇集所有参考的内容，外加小科普 IPv6的查看等基本用法，希望能帮到有需要的人。

### 更新记录

> 2018-04-18
> 修正开启ipv6内核支持的链接与操作方式；添加域名商解析AAAA记录的步骤