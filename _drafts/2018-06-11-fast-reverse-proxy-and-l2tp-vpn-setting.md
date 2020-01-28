---
layout: post
title: 内网穿透与VPN
date: 2018-06-11
tags: [HowTo]
feature-img: https://images.pexels.com/photos/1089438/pexels-photo-1089438.jpeg?cs=srgb&dl=abstract-access-close-up-1089438.jpg&fm=jpg
---




鉴于公司有使用内网的VPN的需求，折腾一个下午，过往的 `VPN` 配置较为陈旧，适应不了当下的要求。公司有动态公网IP，可以临时连接，但这不是长久之计。所幸朋友推荐内网穿透的神器 `frp` ，啧啧，太强大了。

<!--more-->


## 内网穿透 frp

frp 是一个可用于内网穿透的高性能的反向代理应用，支持 tcp, udp, http, https 协议。 官方中文文档在 [这里](https://github.com/fatedier/frp/blob/master/README_zh.md)，项目主页在 [这里](https://github.com/fatedier/frp)。

官方文档写得非常详细，建议移步阅读。其区分 `服务端` 与 `客户端` ，以下是简化的部署步骤。

- 服务端

到 [版本页](https://github.com/fatedier/frp/releases) 进行下载，此处拿最新的。服务端需要搭建在一台具有 `固定公网IP` 的服务器。

```shell
# 自行选择相应的平台
$ cd /data/ && wget https://github.com/fatedier/frp/releases/download/v0.20.0/frp_0.20.0_linux_amd64.tar.gz

# 解压出来
$ tar -xf frp_0.20.0_linux_amd64.tar.gz
$ cd frp_0.20.0_linux_amd64

# 修改 frps.ini 服务端设置
$ vim frps.ini
[common]
bind_port = 7000
privilege_token = 112233445566
privilege_mode = true
```

编辑启动服务文件：`/etc/systemd/system/frps.service`

```shell
[Unit]
Description=frps daemon

[Service]
Type=simple
ExecStart=/data/frp_0.20.0_linux_amd64/frps -c /data/frp_0.20.0_linux_amd64/frps.ini

[Install]
WantedBy=multi-user.target
```

启动服务即可

```shell
$ sudo systemctl start frps.service
$ sudo systemctl enable frps.service
```

- 客户端

客户端即是公司内部的服务器，同样选择在一台 **ubuntu** 系统来部署。

下载并解压 ，修改配置

```shell
$ cd /data/ && wget https://github.com/fatedier/frp/releases/download/v0.20.0/frp_0.20.0_linux_amd64.tar.gz

# 解压出来
$ tar -xf frp_0.20.0_linux_amd64.tar.gz
$ cd frp_0.20.0_linux_amd64

# 修改 frpc.ini 客户端设置
$ vim frpc.ini
[common]
server_addr = x.x.x.x # 服务端的公网IP
server_port = 7000
privilege_token = 112233445566
```

同样，编辑服务启动文件：`/etc/systemd/system/frpc.service`

```shell
[Unit]
Description=frpc daemon

[Service]
Type=simple
ExecStart=/data/frp_0.20.0_linux_amd64/frpc -c /data/frp_0.20.0_linux_amd64/frpc.ini

[Install]
WantedBy=multi-user.target
```

启动服务即可

```shell
$ sudo systemctl start frps.service
$ sudo systemctl enable frps.service
```

到此，服务端 与 客户端的配置已经完成。

> 注意，服务端的防火墙要允许7000端口访问，如果有额外开其他端口，也要一并放行。

## 测试

假设从外网连接 公司内网 客户端的 **ubuntu** 服务器，则可以添加以下设置。

```shell
# 客户端配置文件
$ vim /data/frp_0.20.0_linux_amd64/frpc.ini
[common]
server_addr = x.x.x.x # 服务端的公网IP
server_port = 7000
privilege_token = 112233445566

# 以下是新添加的
[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 9167
```

重启客户端服务

```shell
$ sudo systemctl restart frpc.service
```

接着从外网访问即可。

```shell
$ ssh -p 9167 user@x.x.x.x
```

## VPN设置

借助 [docker-ipsec-vpn-server](https://github.com/hwdsl2/docker-ipsec-vpn-server) 这个项目来实现。

在客户端的服务器上安装 `docker` ，如果 *docker* 不太记得，可以参考 [这里](/2018/05/15/docker-summary/)；使用天朝内的源。

拉取镜像

```shell
$ docker pull hwdsl2/ipsec-vpn-server
```

创建环境变量

```shell
# 假设服务放在 /srv 目录
$ sudo mkdir -pv /srv/vpn

$ sudo vim /srv/vpn/vpn.env
VPN_IPSEC_PSK=your_ipsec_pre_shared_key # 共享密钥
VPN_USER=your_vpn_username
VPN_PASSWORD=your_vpn_password
```

创建 docker 容器

```shell
$ docker run \
    --name ipsec-vpn-server \
    --env-file /srv/vpn/vpn.env \
    --restart=always \
    -p 500:500/udp \
    -p 4500:4500/udp \
    -v /lib/modules:/lib/modules:ro \
    -d --privileged \
    hwdsl2/ipsec-vpn-server
```

在服务端的机器上允许 `500,4500` 端口的访问。

```shell
# 假设是 ubuntu 系统 
$ sudo ufw allow proto udp from any to any port 500,4500 comment 'l2tp vpn'
```

接着在手机上测试下，类型选择 `L2TP/IPSec PSK`，填写 *公网IP*，*共享密钥*，*用户名* 和 *密码* 。

## 多用户账户支持

想让上面的VPN支持多用户，需要进入容器内进行设置，官方说明在 [这里](https://github.com/hwdsl2/docker-ipsec-vpn-server/issues/5) 。

```shell
$ docker exec -it ipsec-vpn-server /bin/bash
```

在容器内安装文本编辑软件 ，建议使用 `nano` ，不懂的可以直接装 `vim`

```shell
# 在容器内执行
$ apt-get update && apt-get install nano
```

需要编辑两个文件，详细说明在 [这里](https://github.com/hwdsl2/setup-ipsec-vpn/blob/master/docs/manage-users-zh.md) ，分别是 `/etc/ppp/chap-secrets` 

```shell
# 注意是容器内的文件噢
$ nano /etc/ppp/chap-secrets
"test" l2tpd "test" *

# "你的VPN用户名1"  l2tpd  "你的VPN密码1"  *

```

接着是 `/etc/ipsec.d/passwd `

```shell
# 注意是容器内的文件噢
$ nano vim /etc/ipsec.d/passwd
你的VPN用户名1:你的VPN密码1的加盐哈希值:xauth-psk
```

其中 **加盐哈希值** 可以通过以下命令获取，可以在别的地方生成

```shell
# 以下命令的输出为：你的VPN密码1的加盐哈希值
openssl passwd -1 '你的VPN密码1'
```

注释容器中 `/opt/src/run.sh` 文件中的以下几行，令上述修改生效

```shell
# Create VPN credentials
cat > /etc/ppp/chap-secrets <<EOF
# Secrets for authentication using CHAP
# client  server  secret  IP addresses
"$VPN_USER" l2tpd "$VPN_PASSWORD" *
EOF

VPN_PASSWORD_ENC=$(openssl passwd -1 "$VPN_PASSWORD")
cat > /etc/ipsec.d/passwd <<EOF
$VPN_USER:$VPN_PASSWORD_ENC:xauth-psk
EOF
```

编辑完毕后，退出容器，重启该容器即可。

```shell
$ docker restart ipsec-vpn-server
```

客户端连接的配置，参考 [IPsec/L2TP](https://github.com/hwdsl2/setup-ipsec-vpn/blob/master/docs/clients-zh.md) 和 [IPsec/XAuth](https://github.com/hwdsl2/setup-ipsec-vpn/blob/master/docs/clients-xauth-zh.md) ，其中 **windows** 的设置略为蛋疼。

## 小结

VPN的实现，略有点绕，它毕竟是免费，比花钱买 **花生壳** 之类的强太多了。对于具有 `固定公网IP` 的服务器这点而言，是付费的。

`frp` 的功能异常强大，完全在家里挂一台 `树莓派` 之类的来玩了，嘿嘿！
