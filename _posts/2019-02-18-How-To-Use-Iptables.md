---
title: How To Use Iptables
layout: post
date: 2019-02-18
tags: [HowTo]
feature-img: "https://images.pexels.com/photos/207142/pexels-photo-207142.jpeg?cs=srgb&dl=brick-wall-bricks-stone-207142.jpg&fm=jpg"
---

Iptables is a very important skill that must be learned by every system administrator.

<!--more-->

## Basic Usage

Enable iptables on centos 7 as root.

```shell
$ systemctl stop firewalld

$ yum install -y iptables-servers
$ systemctl start iptables
$ systemctl enable iptables
```

There are 3 tables including `NAT`, `FILTER`, `MANGLE` and 5 chains including `PREROUTING`, `POSTROUTING`, `INPUT`, `OUTPUT` , `REDIRECT` . Different chains depends on the specific table.

Iptables default configuration file is `/etc/sysconfig/iptables`, anytime you restart the service, it will read iptables rules from the file. Let's see the default rules looks like.

```shell
# /etc/sysconfig/iptables

*nat
:PREROUTING ACCEPT [0:0]     # default policy is ACCEPT
:INPUT ACCEPT [0:0]          # default policy is ACCEPT
:OUTPUT ACCEPT [0:0]         # default policy is ACCEPT
:POSTROUTING ACCEPT [0:0]    # default policy is ACCEPT

COMMIT


*filter
:INPUT ACCEPT [0:0]    # default policy is ACCEPT
:FORWARD ACCEPT [0:0]  # default policy is ACCEPT
:OUTPUT ACCEPT [0:0]   # default policy is ACCEPT
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT

#-A INPUT -j REJECT --reject-with icmp-host-prohibited
#-A FORWARD -j REJECT --reject-with icmp-host-prohibited

COMMIT
```

Here are some examples of iptables commands usages as root.

```shell
# print filter table all rules
$ iptable -t filter -nvL

# print filter table INPUT rules
$ iptable -t filter -nvL INPUT

# default table is filter and show rules num
$ iptable -nvL --line

# set filter table FORWARD chains's default policy is REJECT
$ iptable -P FORWARD REJECT

# allow port 8000
$ iptable -t filter -A INPUT -p tcp -m state --state NEW -m tcp --dport 8000 -j ACCEPT

# delete a rule depend on the rule number
$ iptables -t filter -nvL INPUT --line
$ iptables -t filter INPUT -D 2  # 2 is the rule's number
```

## Forward

Forward also very important especially you want to run Linux as a router or gateway. Let's do some experiments to know more in details. Assuming you have deployed the network environment as follow.

>  Do these experiment on your local machines and run commands as root !!!

```shell
RouterA       # 3.1-eth0
├── PC1       # 3.10
├── PC2       # 3.20
└── RouterB/PC3    # 3.30-enp0s3/8.1-enp0s8
    ├── PC3        # 8.2
    └── PC4        # 8.3
```

Now we want **PC2** can communicate with **PC4** each other through **PC3** .

**PC3** basic requirement settings.

```shell
# enable kernel ip_forward
$ echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.conf
$ sysctl -p

# if the result is 1, meaning you turn on the ip_forward
$ cat /proc/sys/net/ipv4/ip_forward

# use deault iptables rules showed before.
# set filter table FORWARD chain's default policy is ACCEPT
$ systemctl restart iptable
$ iptable -t filter FORWARD -P ACCEPT
```

**PC2** settings

```shell
# add a new route rule
$ route add -net 192.168.8.0/24 gw 192.168.3.30
```

Now **PC2** and **PC4** can ping each other, but let's add a new rule on **PC3** .

```shell
$ iptable -t filter -A FORWARD -j REJECT
```

They can't ping each other this time, now we need to add specific rules which ports was allowed to  communicate. Assume we want the 8000 port.

```shell
# pc4, disable iptables temporarily
$ python -m SimpleHTTPServer 8000 &

# pc2, disable iptables temporarily
$ python -m SimpleHTTPServer 8000 &
```

Allow **PC2** telnet **PC4** 8000 port

```shell
# pc3 setting
$ iptables -t filter -I FORWARD -s 192.168.3.0/24 -p tcp --dport 8000 -j ACCEPT

# remeber to add return rules
$ iptables -t filter -I FORWARD -s 192.168.8.0/24 -p tcp --sport 8000 -j ACCEPT
```

Allow **PC4** telnet **PC2** 8000 port

```shell
# pc3 setting
$ iptables -t filter -I FORWARD -s 192.168.8.0/24 -p tcp --dport 8000 -j ACCEPT

# remeber to add return rules
$ iptables -t filter -I FORWARD -s 192.168.3.0/24 -p tcp --sport 8000 -j ACCEPT
```

The return rules can replace by this rule.

```shell
# pc3
$ iptable -t filter -I FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT
```

- `Static Route`

If you want subnet **192.168.3.0/24** communicate with **192.168.8.0/24** each other.

```shell
# RouterA
$ route add -net 192.168.8.0/24 gw 192.168.3.30 dev eth0
```

## NAT

Assume *192.168.3.0* is WAN and *192.1168.8.0* is LAN. This time **PC3** work as a router, *Router B*. Don't forget the basic requirement settings.

- `SNAT`

**PC2** only knows the request comes from *Router B*, when **PC4** browse **PC2** 8000 port.

```shell
# Router B or PC3
$ iptables -t nat -A POSTROUTING -s 192.168.8.0/24 -j SNAT --to-source 192.168.330
```

If Router B is not a static ip, such as PPPOE client used dynamic ip, then use the special action, `MASQUERADE`.        
```shell
# Router B or PC3
$ iptables -t nat -A POSTROUTING -s 192.168.8.0/24 -o enp0s3 -j MASQUERADE
```

- `DNAT`

Browse the web server behind Router B throng Router B ip address. Assume **PC4** web server created by python is allow browse on **PC2** throng *192.168.3.30:80* .

```shell
# Router B or PC3
$ iptable -t nat -A PREROUTING -d 192.168.3.30 -p tcp --dport 80 -j DNAT --to-destination 192.168.8.3:8000
```

PPPOE dynamic ip

```shell
# Router B or PC3
$ iptable -t nat -A PREROUTING -i enp0s3 -p tcp --dport 80 -j DNAT --to-destination 192.168.8.3:8000
```

## Others

Only use on local port redirect.  80 -> 8000

```shell
# PC4
$ iptable -t filter -A REDIRECT -m state --state NEW -p tcp --dport 80 -j REDIRECT --to-ports 8000
$ iptables -t filter -A INPUT -m state --state NEW -p tcp -m multiport 80,8000 -j ACCEPT
```

Log all 1194 udp port activity.

```shell
# /etc/rsyslog.conf
$ echo 'kern.warning /var/log/iptables.log' >> /etc/rsyslog.conf
$ systemctl restart rsyslog

# iptables rule
$ iptable -t filter -m state --state NEW -p udp -m udp --dport 1194 -j LOG
```

Allow port 80 communicate during 8:00 a.m. to 20:00 p.m.

```shell
$ iptables -t filter -p tcp -m state NEW -m tcp --dport 80 -m time --datestart 08:00 --datestop 20:00 -j ACCEPT
```

## Summary

There is no summary, cause I haven't finish this post yet 😂 