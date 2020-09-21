---
layout: post
title: How To Deploy DNS Master-Slave Service
date: 2019-09-27
tags: HowTo
---

There are some experiment information I should mentioned as blow 

| Server Type | IP Address    | Domain Name          |
| ----------- | ------------- | -------------------- |
| Master      | 192.168.50.11 | master.sparkle.local |
| Slave       | 192.168.50.22 | slave.sparkle.local  |


## Master 

Install required RPM packages.

```shell
$ yum install -y bind bind-utils
```

Modify the main configuration file saved in `/etc/named.conf`

```shell
options {
	# add master ip address
    listen-on port 53 { 127.0.0.1;192.168.50.11; }; 
    #listen-on-v6 port 53 { ::1; };  # disable ipv6 I don't want 
    directory       "/var/named";
    dump-file       "/var/named/data/cache_dump.db";
    statistics-file "/var/named/data/named_stats.txt";
    memstatistics-file "/var/named/data/named_mem_stats.txt";
    # define client ip range which can use current dns service
    allow-query     { localhost;0.0.0.0/0; };
    # transfer to the slave server
    allow-transfer     { 192.168.50.22;};

    recursion yes;

    dnssec-enable yes;
    dnssec-validation yes;

    /* Path to ISC DLV key */
    bindkeys-file "/etc/named.iscdlv.key";

    managed-keys-directory "/var/named/dynamic";

    pid-file "/run/named/named.pid";
    session-keyfile "/run/named/session.key";

};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "." IN {
        type hint;
        file "named.ca";
};

# define master forward zone configuration
zone "sparkle.local" IN {             
    type master;                       
    file "sparkle.local.zone";
    notify yes;
    allow-update { none; };            
};                                 

# define master reverse zone configuration
zone "50.168.192.in-addr.arpa" IN {
    type master;                       
    file "50.168.192.zone";
    notify yes;
    allow-update { none; };            
};                                 

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

Create forward zone configuration saved in  `/var/named/sparkle.local.zone `

```shell
$TTL 86400
@   IN  SOA     master.sparkle.local. root.sparkle.local. (
        2011071001  ;Serial
        3600        ;Refresh
        1800        ;Retry
        604800      ;Expire
        86400       ;Minimum TTL
)
@       IN  NS          master.sparkle.local.
@       IN  NS          slave.sparkle.local.
@       IN  A           192.168.50.11
@       IN  A           192.168.50.22
master       IN  A   192.168.50.11
slave    IN  A   192.168.50.22
```

Create reverse zone configuration saved in `/var/named/50.168.192.zone `

```shell
$TTL 86400
@   IN  SOA     master.sparkle.local. root.sparkle.local. (
        2011071001  ;Serial
        3600        ;Refresh
        1800        ;Retry
        604800      ;Expire
        86400       ;Minimum TTL
)
@       IN  NS          master.sparkle.local.
@       IN  NS          slave.sparkle.local.
@       IN  PTR         sparkle.local.
master       IN  A   192.168.50.11
slave    IN  A   192.168.50.22
```

Check if the main configuration file is correct or not.

```shell
# it's right for nothing output
$ named-checkconf /etc/named.conf

# It's right for the output contains OK, otherwise you should check the zone config again
$ named-checkzone sparkle.local /var/named/sparkle.forward
$ named-checkzone sparkle.local /var/named/sparkle.reverse
```

Start and enable the named service.

```shell
$ systemctl start named
$ systemctl enable named
```

Update network interface configuration to use current DNS address.

```shell
$ vim /etc/sysconfig/network-scripts/ifcfg-ens192
# add these lines
DNS1=192.168.50.11
DNS2=192.168.50.22
```

Update `/etc/resolv.conf` to use current DNS address.

```shell
$ cat /etc/resolv.conf
nameserver   192.168.50.11 #master
nameserver   192.168.50.22 #slave
```

Restart the network service

```shell
$ systemctl restart network
```

Allow the DNS service default port 53 through iptables. 

```shell
# iptables
$ iptables -I INPUT -m state --state NEW -p tcp --dport 53 -j ACCEPT
$ iptables -I INPUT -m state --state NEW -p udp --dport 53 -j ACCEPT

# firewalld
$ firewall-cmd --permanent --add-port=53/tcp
$ firewall-cmd --permanent --add-port=53/udp
$ firewall-cmd --reload
```


## Slave

Install required RPM packages.

```shell
$ yum install -y bind bind-utils
```

Modify the main configuration file saved in `/etc/named.conf`

```shell
options {
	# add slave ip address
    listen-on port 53 { 127.0.0.1;192.168.50.22; }; 
    #listen-on-v6 port 53 { ::1; };  # disable ipv6 I don't want 
    directory       "/var/named";
    dump-file       "/var/named/data/cache_dump.db";
    statistics-file "/var/named/data/named_stats.txt";
    memstatistics-file "/var/named/data/named_mem_stats.txt";
    # define client ip range which can use current dns service
    allow-query     { localhost;0.0.0.0/0; };

    recursion yes;

    dnssec-enable yes;
    dnssec-validation yes;

    /* Path to ISC DLV key */
    bindkeys-file "/etc/named.iscdlv.key";

    managed-keys-directory "/var/named/dynamic";

    pid-file "/run/named/named.pid";
    session-keyfile "/run/named/session.key";

};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "." IN {
        type hint;
        file "named.ca";
};

# define slave forward zone configuration
zone "sparkle.local" IN {
    type slave;
    file "slaves/sparkle.local.zone";
    masters { 192.168.50.11; };
};

# define slave reverse zone configuration
zone "50.168.192.in-addr.arpa" IN {
    type slave;
    file "slaves/50.168.192.zone";
    masters { 192.168.50.11; };
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

Check if the main configuration file is correct or not.

```shell
# it's right for nothing output
$ named-checkconf /etc/named.conf
```

Start and enable named service

```shell
$ systemctl start named
$ systemctl enable named
```

There is no need to create zone configurations manually at this time, which will be sync by master named service. But you should check the zone configurations exist or not after about 5 minutes , maybe less.

Update network interface configuration to use current DNS address.

```shell
$ vim /etc/sysconfig/network-scripts/ifcfg-ens192
# add these lines
DNS1=192.168.50.11
DNS2=192.168.50.22
```

Update `/etc/resolv.conf` to use current DNS address.

```shell
$ cat /etc/resolv.conf
nameserver   192.168.50.11 #master
nameserver   192.168.50.22 #slave
```


## Client

Assuming the client system is centos 7. There are 3 ways to check if the DNS service is correct or not.  

Make sure you have updated network interface configuration and resolve configuration.

- Ping

The easiest way to check domain name. 

```shell
$ ping master.sparcle.local
Reply from 192.168.50.11: bytes=32 time=1ms TTL=126
```

- nslookup

The flexible way you can specify which DNS address you want to use.

```shell
# use default dns address to check domain name
$ nslookup master.sparcle.local
Server:         192.168.50.11
Address:        192.168.50.11#53

Name:   master.sparcle.local
Address: 192.168.50.11

# use slave dns address to check domain name
$ nslookup master.sparcle.local 192.168.50.22
Server:         192.168.50.22
Address:        192.168.50.22#53

Name:   master.sparcle.local
Address: 192.168.50.11
```

- dig

The way you can get full information about domain name .

```shell
$ dig master.sparcle.local
; <<>> DiG 9.9.4-RedHat-9.9.4-61.el7 <<>> vcenter.ztrx.gz
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 56346
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;master.sparcle.local.               IN      A

;; ANSWER SECTION:
master.sparcle.local.        86400   IN      A       192.168.50.11

;; AUTHORITY SECTION:
ztrx.gz.                86400   IN      NS     master.sparcle.local.

;; ADDITIONAL SECTION:
master.sparcle.local.         86400   IN      A       192.168.50.11

;; Query time: 0 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Mon Nov 18 10:23:49 CST 2019
;; MSG SIZE  rcvd: 97
```

## Sync

Assume you want to add a new A record to DNS service like this 

```shell
$ ping test.sparcle.local
Reply from 192.168.50.33: bytes=32 time=1ms TTL=126
```

Add this line to both forward zone configuration file and reverse zone configuration file in master server.

```shell
slave    IN  A   192.168.50.33
```

The most `important` thing is that change the `Serial` value and make it larger than before, or the slave server can not get update record from master server.

