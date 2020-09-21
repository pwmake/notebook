---
layout: post
title: How to deploy local http yum repository
date: 2019-09-25
tags: HowTo
---

Installing some RPM packages without WAN network can be painful. Why not deploy a local http yum repository to manage your rpm packages.

Assume we have two host just like this 

1. CentOS 7 Server - 1.2.3.4
2. CentOS 7 Client - 1.2.3.5

## Server

Mount CentOS 7 DVD iso into the server .

```shell
$ mkdir -pv /media/CentOS

$ mount /dev/cd-rom /media/CentOS
```

Create local yum repository 

```shell
# create local repository directory
$ mkdir -pv /data/mylocal

# sync rpm packages
$ cp -ap /media/CentOS/* /data/mylocal

# create local repository info
$ yum -y install createrepo --disablerepo='*' --enablerepo='c7-media'
$ createrepo /data/mylocal
```

Create local yum repository configuration and Test it

```shell
$ vim /etc/yum.repos.d/mylocal.repo
[mylocal]
name=mylocal
baseurl=file://data/mylocal
enabled=1
gpgcheck=0

$ yum clean all && yum repolist all

# test the repository
$ yum search vsftp --disablerepo='*' --enablerepo=mylocal
```

Add a virtual host configuration in nginx.

```shell
$ yum -y install nginx

$ vim /etc/nginx/conf.d/mylocal.conf
server {
    listen 8090;
    server_name your_server_name;
    root /data/mylocal;
    location / {
    	autoindex on;  # Make Sure this line exist !
    }
}

$ sudo nginx -t 
$ systemctl reload nginx
$ systemctl enable nginx
```

Consider the only repository is `mylocal` , you can delete other `*.repo` saved in `/etc/yum.repos.d` , which make **mylocal** as the default repository. 

Allow the port you defined in nginx virtual host through iptables.

```shell
# port 8090 defined in nginx 
$ iptable -I INPUT -m state --state NEW -p tcp --dport 8090 -j ACCEPT 
```

Remember to umount the DVD iso .

```shell
$ umount /media/CentOS
```

## Client

Add a new repository configuration saved in `/etc/yum.repos.d/mylocal` .

```shell
$ vim /etc/yum.repos.d/mylocal.repo
[mylocal]
name=mylocal
baseurl=http://192.168.50.11:8090    
enabled=1
gpgcheck=0
```

Set **mylocal** as default repository and update yum cache.

```shell
$ mv CentOS-*.repo /tmp

$ yum clean all && yum makecache
```

Try to install a rpm package whether the repository run correctly or not.

```shell
$ yum install w3m -y
```

## Update Repository

You need to update repository manually, when you want to add some new packages. 

Assume you want to add package **vsftpd** . The package dependence could be solved by *yum* command. 

```shell
# execute these command in a centos contain WAN network

# clear cache packages
$ find /var/cache/yum/x86_64/7 -name "*.rpm" -exec rm -f {} \;

# download vsftpd package
$ yum install vsftpd
	Resolving Dependencies                                                 
	--> Running transaction check                                           
	---> Package vsftpd.x86_64 0:3.0.2-25.el7 will be installed             
	--> Finished Dependency Resolution                                                                                                   
	Dependencies Resolved                    
  ==============================================================
	 Package         Arch         Version      Repository       Size 
	=====================================================================
	Installing:              
	 vsftpd          x86_64        3.0.2-25.el7       base         171 k 
	Transaction Summary        
  ======================================================================
	Install  1 Package     
	Total download size: 171 k     
	Installed size: 353 k     
	Is this ok [y/d/N]: d   # Choose d !!!  
	Background downloading packages, then exiting:  
	vsftpd-3.0.2-25.el7.x86_64.rpm               | 171 kB  00:00:06 
	exiting because "Download Only" specified 
	
# tar all of the related packages 
$ mkdir -pv /tmp/vsfptd
$ find /var/cache/yum/x86_64/7 -name "*.rpm" -exec cp {} /tmp/vsftpd \;
$ cd /tmp/vsftpd && tar -jcf vsftpd_rpms.tar.bz2 *.rpm
```

Upload the **vsftpd_rpms.tar.bz2** to server , which means the *192.168.50.11*, then create repository once again.

```shell
# assume vsftpd_rpms.tar.bz2 saved in /usr/local/src

$ tar -xf /usr/local/src/vsftpd_rpms.tar.bz2 -C /data/mylocal/Packages
$ createrepo /data/mylocal
```

BOOM ! Well Done ! You do a great job. 

