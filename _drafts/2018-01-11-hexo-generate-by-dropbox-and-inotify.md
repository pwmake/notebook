---
layout: post
title: dropbox + inotify自动更新hexo博客
date: 2018-01-11
tags: [Skill]
feature-img: "https://images.pexels.com/photos/436781/pexels-photo-436781.jpeg?cs=srgb&dl=access-accessories-blur-436781.jpg&fm=jpg"
---


一种既无后台， 又能方便随时随地管理 hexo 博客的方法，缺点是图片上传不能一并处理，嘿嘿！


<!--more-->


## 安装dropbox

Linux 直接使用命令行安装，瘟系统 和 苹果 可以直接安装的，运行后再设置代理即可。

```bash
# 通用是默认装在家目录
$ cd ~ && wget -O - "https://www.dropbox.com/download?plat=lnx.x86_64" | tar xzf -
# 如果下载不来，则在自己的VPS上下载好，把 ~/.dropbox-list 下载到本地

# 运行 dropbox
$ ~/.dropbox-dist/dropboxd
```

首次运行时，需要访问一个链接，设置绑定到当前机器，按指引操作。

接着再下载管理脚本 [dropbox.py](https://www.dropbox.com/download?dl=packages/dropbox.py) ，进行日常操作使用。可以这样设置

```bash
$ echo 'alias drop="/home/your_name/dropbox.py"' >> ~/.bashrc

$ source ~/.bashrc

# 查看命令帮助
$ drop help 

# 设置代理
$ drop proxy manual socks5 x.x.x.x port username[可选] password[可选]

# 过滤不同步的目录，必须在 运行状态下添加，写绝对路径噢
$ drop exclude add /home/your_name/Dropbox/xxx/xxx  
```

## 安装hexo

重新安装，或者将现有的hexo目录丢到 dropbox 所在目录即可。

```shell
$ cd /home/your_name/Dropbox 
$ npm install hexo-cli -g
$ hexo init blog
$ npm server
```

记得把 `node_modules` 目录取消同步，否则几千个文件，同步上去没啥意义。

## 监听文章

借助 `inotify` 来实现 监听`_posts` 目录下的 *markdown* 文件变化，重新生成静态网站文件。

```shell
$ sudo apt install inotify-tools
```

撰写监听脚本 `inotify.sh`

```shell
$ cat > inotify.sh << EOF
#! /bin/bash

# 如果是nvm管理npm的，则需要执行下列环境变量
export NVM_DIR="/home/jeff/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"

/usr/bin/inotifywait -rmq  -e modify,create,delete,attrib /home/your_name/Dropbox/hexo/source/_posts/ |  while read  event
do
    /home/your_name/Dropbox/hexo && hexo g
done

```

~~借助 `nohup` 来后台执行~~ 。

nohup 并不能保证永远在开启，并且设置开机启动也不太友好；弃用之。

## 开机启动

- 开机启动 dropbox

借助 `supervisor` 来管理 *dropbox* 的开机启动，只保证开机启动，日常管理，仍然在家目录，执行 *dropbox.py* 脚本来实现，上面有讲过的已经。

创建配置文件： /etc/supervisor/conf.d/dropbox.conf

```shell
[program:dropbox]
command=/home/jeff/dropbox.py start
directory=/home/jeff
user=jeff
autostart=true
autorestart=false
redirect_stderr=true
stdout_logfile=/data/logs/dropbox.log
```

重启 supervisor 服务「此服务默认开机启动」

```shell
sudo service supervisor restart
```

- 开机启动监听脚本

借助 `systemd` 来管理 监听脚本 `inotify.sh`  。

创建服务的配置文件：*/lib/systemd/system/hexo.service*

```shell
[Unit]
Description=Hexo Auto Generation

[Service]
ExecStart=inotify.sh # 写脚本的绝对路径
Restart=always

[Install]
WantedBy=multi-user.target
```

开启服务

```shell
sudo systemctl start hexo
```

设置开机启动

```shell
sudo systemctl enable hexo
```

### 参考链接

> [Dropbox as a systemd service](https://bbkane.github.io/2017/08/04/Dropbox-as-a-systemd-service.html)
>
> [阮一峰 Systemd 入门教程：实战篇](http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-part-two.html)

