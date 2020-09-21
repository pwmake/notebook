---
layout: post
title: How To Load Apache Proxy Module in Qnap
date: 2020-03-14
banner: default
tags: HowTo
---


Log in to the Qnap as administrator via SSH. 

To View the list of avaliable Apache modules, go to 

```shell
/usr/local/apache/modules
```

To view those modules have already load, open this file

```shell
/etc/config/apache/extra/apache-default-modules.conf
```

You should add proxy modules to the main configuration of Apache, showed as below,  not the `apache-default-modules.conf`

```shell
/etc/config/apache/apache.conf
```

Just append these lines into the conf 

```shell
# Added by xxx, xx/xx/2020 23:21:00
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_connect_module modules/mod_proxy_connect.so
LoadModule proxy_http_module modules/mod_proxy_http.so
```

Create a new virtual host configuration through qnap web UI , by going to 

> **Main Menu** > **Control Panel** > **Applications** > **Web Server** > **Virtual Host** > **Create New Virtual Host**.

Fill in your virtual host information as you like in the correct position, such as 

```shell
# hostname 
test.com

# port
80
```

Now open the configuration of user vhost , `/etc/config/apache/extra/httpd-vhosts-user.conf`

```shell
NameVirtualHost *:80


<VirtualHost _default_:80>
        DocumentRoot "/share/Web"
</VirtualHost>

# The vhost your added before
<VirtualHost *:80>
        ServerName test.com
        DocumentRoot "/share/Public"
<Directory "/share/Public">
        Options FollowSymLinks MultiViews
        AllowOverride All
        Require all granted
</Directory>
</VirtualHost>
```

Update some options to support proxy module that client can browse web application behind apache whose port is 8080.

```shell
NameVirtualHost *:80


<VirtualHost _default_:80>
        DocumentRoot "/share/Web"
</VirtualHost>
<VirtualHost *:80>
        ServerName test.com
        ProxyRequests off
        <Proxy *>
            #Order allow,deny
            Require all granted
        </Proxy>
        ProxyPass "/" "http://127.0.0.1:8080/"
        ProxyPassReverse "/" "http://127.0.0.1:8080/"
</VirtualHost>
```

At last, restart Apache service.

```shell
 /etc/init.d/Qthttpd.sh restart
```

BOOM!!! 😉 Enjoy yourself. 

