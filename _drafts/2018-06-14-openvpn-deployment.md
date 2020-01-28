---
layout: post
title: OpenVPN的部署与使用
date: 2018-06-14
tags: [HowTo]
feature-img: https://images.pexels.com/photos/1089438/pexels-photo-1089438.jpeg?cs=srgb&dl=abstract-access-close-up-1089438.jpg&fm=jpg
---


继 [上一篇](/2018/06/11/fast-reverse-proxy-and-l2tp-vpn-setting/) 文章之后，实际试用了几天的 **VPN**，遇到几个问题，连接不太稳定，经常过没10分钟断开；有时甚至无法连接，顿感恼火，哎，算了，还是老实点，弄个 `OpenVPN` 得了。

<!--more-->

本文主要参考 [十步搭建OpenVPN，享受你的隐私生活](https://linux.cn/article-3706-1.html)，并增加 `双重认证` 与 `吊销处理` 。

## 服务端

服务器环境为 **ubuntu 16.04**

安装必要的软件包

```shell
$ sudo apt update
$ sudo apt install -y openvpn easy-rsa dnsmasq
```

生成 `CA` 证书 和 私钥，放置在 目录 `/etc/openvpn/easy-rsa/keys/{ca.crt,ca.key}`

```shell
$ sudo mkdir /etc/openvpn/easy-rsa
$ sudo cp -r /usr/share/easy-rsa/* /etc/openvpn/easy-rsa

# 预设定证书的变量
$ sudo vim /etc/openvpn/easy-rsa/var
export KEY_COUNTRY="CN"
export KEY_PROVINCE="BJ"
export KEY_CITY="Beijing"
export KEY_ORG="Who knows"
export KEY_EMAIL="nobody@gmail.com"
export KEY_CN="anything"
export KEY_NAME="zshmobi"
export KEY_OU="anything"
export KEY_ALTNAMES="zshmobi"

# 切换至 root 用户执行
$ cd /etc/openvpn/easy-rsa
$ source vars
$ ./clean-all
$ ./build-ca
```

生成 **OpenVPN** 服务器的 *证书* 和 *私钥* ，假定叫 `zshmobi`，同样放在 `/etc/openvpn/easy-rsa/keys`

```shell
$ cd /etc/openvpn/easy-rsa
$ ./build-key-server zshmobi
# 同样也有两次输入 y 的过程
```

接着生成 `Diffie-Hellman` ，文件为 `/etc/openvpn/easy-rsa/dh2048.pem`

```shell
$ cd /etc/openvpn/easy-rsa
$ ./build-dh
```

复制服务端配置模板

```shell
# 以root用户执行
$ cd /etc/openvpn
$ cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz .
$ gunzip -d server.conf.gz
$ mv server.conf zshmobi-server.conf
```

自定义服务端配置，文件名为：`/etc/openvpn/zshmobi-server.conf`

```shell
# 完整的配置如下
$ vim zshmobi-server.conf
port 1194
proto udp
dev tun
# 绝对路径的
ca /etc/openvpn/easy-rsa/keys/ca.crt
cert /etc/openvpn/easy-rsa/keys/zshmobi-server.crt
dh /etc/openvpn/easy-rsa/keys/dh2048.pem
server 10.8.0.0 255.255.255.0
ifconfig-pool-persist ipp.txt
push "route 192.168.3.0 255.255.255.0"
push "redirect-gateway def1 bypass-dhcp"

# 客户端连接后使用的DNS
push "dhcp-option DNS 114.114.114.114"
client-to-client
duplicate-cn
keepalive 10 120
comp-lzo
persist-key
persist-tun
status /var/log/openvpn-status.log
# 主要日志
log /var/log/openvpn.log
verb 3

# 脚本验证用户密码
script-security 3 system
auth-user-pass-verify /etc/openvpn/checkpsw.sh via-env
username-as-common-name

# 吊销某个用户的证书，使其无法登录，下面会详细讲
# crl-verify /etc/openvpn/easy-rsa/keys/test01-crl.pem
```

用户密码验证脚本：`/etc/openvpn/checkpsw.sh`

```shell
#!/bin/sh
###########################################################
# checkpsw.sh (C) 2004 Mathias Sundman <mathias@openvpn.se>
#
# This script will authenticate OpenVPN users against
# a plain text file. The passfile should simply contain
# one row per user with the username first followed by
# one or more space(s) or tab(s) and then the password.

# 可自定义以下变量
PASSFILE="/etc/openvpn/psw-file"
LOG_FILE="/etc/openvpn/openvpn-password.log"
TIME_STAMP=`date "+%Y-%m-%d %T"`

###########################################################

if [ ! -r "${PASSFILE}" ]; then
  echo "${TIME_STAMP}: Could not open password file \"${PASSFILE}\" for reading." >> ${LOG_FILE}
  exit 1
fi

CORRECT_PASSWORD=`awk '!/^;/&&!/^#/&&$1=="'${username}'"{print $2;exit}' ${PASSFILE}`

if [ "${CORRECT_PASSWORD}" = "" ]; then 
  echo "${TIME_STAMP}: User does not exist: username=\"${username}\", password=\"${password}\"." >> ${LOG_FILE}
  exit 1
fi

if [ "${password}" = "${CORRECT_PASSWORD}" ]; then 
  echo "${TIME_STAMP}: Successful authentication: username=\"${username}\"." >> ${LOG_FILE}
  exit 0
fi

echo "${TIME_STAMP}: Incorrect password: username=\"${username}\", password=\"${password}\"." >> ${LOG_FILE}
exit 1
```

用户账号与密码文件：`/etc/openvpn/psw-file`

```shell
username01 password01
username02 password02
```

启动服务，并查看端口，如果  `1194` 端口起不来，查看下日志以排查问题。

```shell
$ sudo systemctl start openvpn

# 例如这个日志，
$ tail -f /var/log/openvpn.log
Options error: --crl-verify fails with '/etc/openvpn/easy-rsa/keys/*-crl.pem': No such file or directory
Options error: Please correct these errors.
Use --help for more information.
```


日志不报错后，查看下端口是否起来

```shell
$ sudo netstat -anup
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
udp        0      0 0.0.0.0:54149           0.0.0.0:*                           555/dhclient
udp        0      0 0.0.0.0:1194            0.0.0.0:*                           3024/openvpn
udp        0      0 0.0.0.0:53              0.0.0.0:*                           2756/dnsmasq
udp        0      0 0.0.0.0:68              0.0.0.0:*                           555/dhclient
udp6       0      0 :::60622                :::*                                555/dhclient
udp6       0      0 :::53                   :::*                                2756/dnsmasq
```

为 `OpenVPN` 客户端搭建 **DNS** ，`选做` 

```shell
$ sudo vim /etc/dnsmasq.conf
listen-address=127.0.0.1, 10.8.0.1
bind-interfaces

$ sudo systemctl restart dnsmasq
```

开启端口转发，`重要` 。

```shell
$ sudo vim /etc/sysctl.conf
net.ipv4.ip_forward=1
```

增加以下防火墙规则

```shell
$ iptables -A FORWARD -m state --state RELATED,ESTABLISHED -j ACCEPT
$ iptables -A FORWARD -s 10.8.0.0/24 -j ACCEPT
$ iptables -A FORWARD -j REJECT
# 根据实际的网卡名来设定
$ iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
```

希望每次启动时自动添加，可以添加到 `/etc/rc.local` 。

没有添加防火墙规则，即使客户端连接成功，仍无法访问 **内网网络** 与 **外部网络** ，即不可访问任何网站。


## 增加客户端密钥

出于安全性考虑，对每个用户分配独立的密钥，以确保 **日志中有记录可查**、 **单独处理密钥而不影响其他人使用** 。

此处为添加 ==jeff== 的密钥为例子。

```shell
# 登录 192.168.3.33
$ cd /etc/openvpn/easy-rsa
$ source vars
$ ./build-key jeff  # 这里输入待添加人的姓名 
Generating a 2048 bit RSA private key
..+++
......................+++
writing new private key to 'jeff.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [CN]:
State or Province Name (full name) [GD]:
Locality Name (eg, city) [GuangZhou]:
Organization Name (eg, company) [LT]:
Organizational Unit Name (eg, section) [server]:
Common Name (eg, your name or your server's hostname) [jeff]:
Name [zshmobi]:
Email Address [@@zshmobi.com.com]:jeff@@zshmobi.com.com  # 输入自己的邮箱地址

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
Using configuration from /etc/openvpn/easy-rsa/openssl-1.0.0.cnf
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
countryName           :PRINTABLE:'CN'
stateOrProvinceName   :PRINTABLE:'GD'
localityName          :PRINTABLE:'GuangZhou'
organizationName      :PRINTABLE:'LT'
organizationalUnitName:PRINTABLE:'server'
commonName            :PRINTABLE:'jeff'
name                  :PRINTABLE:'zshmobi'
emailAddress          :IA5STRING:'jeff@@zshmobi.com.com'
Certificate is to be certified until Jun 27 09:18:45 2028 GMT (3650 days)
Sign the certificate? [y/n]:y # 输入 y 继续


1 out of 1 certificate requests certified, commit? [y/n]y  # 输入 y 继续
Write out database with 1 new entries
Data Base Updated
```

创建 客户端配置

```shell
# 通过模板来创建
$ cd /etc/openvpn/easy-rsa/keys
$ cp tpl.ovpn jeff.ovpn

# 修改模板内的选项  
$ vim jeff.ovpn
...省略部分...
<ca>
# 对应 ca.crt 文件内容
</ca> 

<cert>
# 对应 jeff.crt 文件内容
</cert> 

<key>
# 对应 jeff.key 文件内容
</key>
```

把得到的 `jeff.ovpn` 客户端配置，放到相应的机器上使用即可。

如果需要重新生成客户端密钥，则再按上面的顺序执行一次即可。


## 禁用客户端密钥

当某个用户不再使用了，需要对其进行 `吊销密钥、证书` 处理，使其不能登录使用 **VPN** 。假设禁用 `jeff` 登录。

生成禁用的 `*-crl.pem` 文件

> `revoke-full` 已作修改，生成 *用户名-crl.pem* 文件；并自动将记录添加到服务端配置文件中。 

```shell
# 192.168.3.33 上操作
$ cd /etc/openvpn/easy-rsa

$ source var
$ ./revoke-full jeff # 后面跟用户名

# 生成 keys/jeff-crl.pem 文件
# 添加 crl-verify /etc/openvpn/easy-rsa/keys/jeff-crl.pem 到 /etc/openvpn/zshmobi-server.conf
```

密钥数据库 `keys/index.txt` 的记录变成如下

```shell
# /etc/openvpn/easy-rsa/keys/index.txt

$ cat keys/index.txt
V       280702071731Z           01      unknown /C=CN/ST=GD/L=GuangZhou/O=LT/OU=@zshmobi.com/CN=zshmobi-server/name=@zshmobi.com/emailAddress=@@zshmobi.com.com

# 开头变成 R，即代表 REVOKED
R       280702072127Z   180705084430Z   02      unknown /C=CN/ST=GD/L=GuangZhou/O=LT/OU=@zshmobi.com/CN=test01/name=@zshmobi.com/emailAddress=test01@@zshmobi.com.com
```

接着重启服务，以便生效

```shell
$ /etc/init.d/openvpn restart
```

## windows客户端

客户端同样也需要一个配置文件，如上述获得的 `jeff.ovpn` 。

**window** 客户端的下载页面在 [这里](https://openvpn.net/index.php/open-source/downloads.html) 。

## 服务端日志查看

登录到 **OpenVPN服务端** 所在机器上  ，执行 `tail -f /var/log/openvpn.log` ，查看日志

```shell
Sat Jun 30 17:43:45 2018 127.0.0.1:40169 TLS: Initial packet from [AF_INET]127.0.0.1:40169, sid=5d4b5e44 eca28c43
# 查看到用户的邮箱
Sat Jun 30 17:43:45 2018 127.0.0.1:40169 VERIFY OK: depth=1, C=CN, ST=GD, L=GuangZhou, O=LT, OU=server, CN=zshmobi, name=zshmobi, emailAddress=jeff@@zshmobi.com.com
Sat Jun 30 17:43:45 2018 127.0.0.1:40169 VERIFY OK: depth=0, C=CN, ST=GD, L=GuangZhou, O=LT, OU=server, CN=jeff, name=zshmobi, emailAddress=jeff@@zshmobi.com.com
Sat Jun 30 17:43:45 2018 127.0.0.1:40169 Data Channel Encrypt: Cipher 'BF-CBC' initialized with 128 bit key
Sat Jun 30 17:43:45 2018 127.0.0.1:40169 WARNING: this cipher's block size is less than 128 bit (64 bit).  Consider using a --cipher with a larger block size.
Sat Jun 30 17:43:45 2018 127.0.0.1:40169 Data Channel Encrypt: Using 160 bit message hash 'SHA1' for HMAC authentication
Sat Jun 30 17:43:45 2018 127.0.0.1:40169 Data Channel Decrypt: Cipher 'BF-CBC' initialized with 128 bit key
Sat Jun 30 17:43:45 2018 127.0.0.1:40169 WARNING: this cipher's block size is less than 128 bit (64 bit).  Consider using a --cipher with a larger block size.
Sat Jun 30 17:43:45 2018 127.0.0.1:40169 Data Channel Decrypt: Using 160 bit message hash 'SHA1' for HMAC authentication
Sat Jun 30 17:43:45 2018 127.0.0.1:40169 Control Channel: TLSv1.2, cipher TLSv1/SSLv3 DHE-RSA-AES256-GCM-SHA384, 2048 bit RSA
Sat Jun 30 17:43:45 2018 127.0.0.1:40169 [jeff] Peer Connection Initiated with [AF_INET]127.0.0.1:40169
Sat Jun 30 17:43:45 2018 jeff/127.0.0.1:40169 MULTI_sva: pool returned IPv4=10.8.0.10, IPv6=(Not enabled)
# 这两行可以看到哪个key登录的
Sat Jun 30 17:43:45 2018 jeff/127.0.0.1:40169 MULTI: Learn: 10.8.0.10 -> jeff/127.0.0.1:40169
Sat Jun 30 17:43:45 2018 jeff/127.0.0.1:40169 MULTI: primary virtual IP for jeff/127.0.0.1:40169: 10.8.0.10
Sat Jun 30 17:43:45 2018 jeff/127.0.0.1:40169 PUSH: Received control message: 'PUSH_REQUEST'
Sat Jun 30 17:43:45 2018 jeff/127.0.0.1:40169 send_push_reply(): safe_cap=940
Sat Jun 30 17:43:45 2018 jeff/127.0.0.1:40169 SENT CONTROL [jeff]: 'PUSH_REPLY,redirect-gateway def1 bypass-dhcp,route 10.8.0.1,topology net30,ping 10,ping-restart 120,ifconfig 10.8.0.10 10.8.0.9' (status=1)
```

查看吊销证书后的用户登录日志

```shell
Thu Jul  5 16:46:19 2018 127.0.0.1:34882 TLS: Initial packet from [AF_INET]127.0.0.1:34882, sid=10b990f5 05041960
Thu Jul  5 16:46:19 2018 127.0.0.1:34882 CRL CHECK OK: C=CN, ST=GD, L=GuangZhou, O=LT, OU=@zshmobi.com, CN=@zshmobi.com, name=@zshmobi.com, emailAddress=@@zshmobi.com.com
Thu Jul  5 16:46:19 2018 127.0.0.1:34882 VERIFY OK: depth=1, C=CN, ST=GD, L=GuangZhou, O=LT, OU=@zshmobi.com, CN=@zshmobi.com, name=@zshmobi.com, emailAddress=@@zshmobi.com.com
Thu Jul  5 16:46:19 2018 127.0.0.1:34882 CRL CHECK FAILED: C=CN, ST=GD, L=GuangZhou, O=LT, OU=@zshmobi.com, CN=test01, name=@zshmobi.com, emailAddress=test01@@zshmobi.com.com (serial 02) is REVOKED
Thu Jul  5 16:46:19 2018 127.0.0.1:34882 TLS_ERROR: BIO read tls_read_plaintext error: error:14089086:SSL routines:ssl3_get_client_certificate:certificate verify failed
Thu Jul  5 16:46:19 2018 127.0.0.1:34882 TLS Error: TLS object -> incoming plaintext read error
Thu Jul  5 16:46:19 2018 127.0.0.1:34882 TLS Error: TLS handshake failed
Thu Jul  5 16:46:19 2018 127.0.0.1:34882 SIGUSR1[soft,tls-error] received, client-instance restarting
```


## [客户端路由设置](https://blog.csdn.net/caizhengwu/article/details/80066338)

**OpenVPN** 默认连接时所有数据都走 VPN 通道，造成所有访问都非常慢。可以手动设置，主要由以下3个参数决定

`route-nopull`：连接后不添加路由，不会有任何网络请求走 *OpenVPN*

`vpn_gateway`：设定上面的参数后，此参数指定哪些IP走VPN

```shell
route 192.168.2.0 255.255.255.0 vpn_gateway
route 10.0.0.0 255.255.255.0 vpn_gateway
```

`net_gateway`：与第二个相反，表示默认全部走VPN时，指定哪些 IP 不走VPN

```shell
max-routes 1000  # 最大值 
route 172.121.0.0. 255.255.0.0 net_gateway
```

通常是 `1+2` 的搭配来使用。

## 小结

对于没固定公网IP的情况，则需要另外制定 `DDNS` ，部分域名商提供接口方便写脚本定时更新；或者使用 `frp` 内网穿透来实现。

> 更新记录
>
> 2018-07-05  增加 `密码+证书` 双重认证，增加 `吊销证书` 处理。
