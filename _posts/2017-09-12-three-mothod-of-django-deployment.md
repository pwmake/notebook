---
layout: post
title: django部署的3种方法
date: 2017-09-12 14:17:57
tags: [Django]
feature-img: "https://images.pexels.com/photos/436781/pexels-photo-436781.jpeg?cs=srgb&dl=access-accessories-blur-436781.jpg&fm=jpg"
---

当我们初始化一个 *django* 项目时，它会自己创建一个 `wsgi.py` ，其提供了 *django* 项目部署时，能够使用任何的 `WSGI`服务。听起来很难懂是吧？无论使用 *apache* 或者是 *nginx* 作 *web* 服务，真正处理请求的是 `wsgi.py` ，可以把它理解为一个桥梁，连接 *web* 服务和 *django* 实际的处理逻辑。对于 `PHPer` ，`wsgi` 相当于 `fastcgi` ，前者服务对象是 `python` ，后者则是 `PHP` ，嘿嘿。

<!--more-->
官方推荐的3种部署方法，分别是：

1. `uWSGI` 与 `Nginx` 
2. `Gunicorn` 与 `Nginx` 
3. `Apache` 与 `mod_wsgi` 

以下分别针对这三种方法做实践讲解，最后再做对比小结。

实验环境为` ubuntu 16.04 系统`，注意！！！ 

## Apache 与 mod_wsgi

最简单快捷的部署方法，适用于纯粹想尝试部署的同学。但是……敲重点了，部署简单不代表操作起来很简单噢。

鉴于操作系统、apache版本的差异性，有以下组合

1. ubuntu + apache2.4 + mod_wsgi ，难度指数 0。
2. CentOS + apache2.2 + python2.7 + mod_wsgi，难度指数中等，详情参见 [CentOS部署django](/post/deploy-django-on-centos-with-apache)。
3. Windows + apache + mod_wsgi，难度指数上天，涉及VC库的编译，详情参见 [windows+mod_wsgi](https://github.com/GrahamDumpleton/mod_wsgi/blob/develop/win32/README.rst) ，此平台的可跳过本节。

此处只讲 难度指数为 0 的方案。

新建一个示例项目

```shell
# 使用虚拟环境
$ sudo pip install virtualenv

# 创建虚拟环境
$ virtualenv /data/django

# 新建项目
$ cd /data/django && source bin/activate
$ django-admin new_site
```

### 基础配置

新建一个名为 `new_site.conf` 的 apache配置，重点内容是

```shell
# 不能包括在 virtualhost 内
WSGIPythonHome /data/django/
WSGIPythonPath /data/django/new_site/

<VirtualHost *:80>
	...
	WSGIScriptAlias / /data/django/new_site/new_site/wsgi.py

	# 允许执行wsgi.py
    <Directory /data/django/new_site/new_site>
        <Files wsgi.py>
            Require all granted
        </Files>
    </Directory>
	...
</VritualHost>
```

注意有两项是不能放在 `<VirtualHost>` 块内的。

### 守护进程模式

亦可以让 `mod_wsgi` 以守护进程的模式运行，则配置 `new_site.conf` 写法变为

```shell
<VirtualHost *:80>
	...
	WSGIScriptAlias / /data/django/new_site/new_site/wsgi.py
	# 增加以下两条
    WSGIDaemonProcess new_site python-home=/data/django python-path=/data/django/new_site
    WSGIProcessGroup new_site

	# 允许执行wsgi.py
    <Directory /data/django/new_site/new_site>
        <Files wsgi.py>
            Require all granted
        </Files>
    </Directory>
	...
</VritualHost>
```

由于 *django* 本身不提供 *web* 服务，静态文件则需要借助 *apache* 或者是 *nginx* 来实现，如在 `new_site.conf` 添加 `static` 的访问

```shell
Alias /media/ /data/django/new_site/media/
Alias /static/ /data/django/new_site/static/

<Directory /data/django/new_site/media>
Require all granted
</Directory>

<Directory /data/django/new_site/static>
Require all granted
</Directory>
```

## Gunicorn 与 Nginx

`Gunicorn` 是一款纯 python 的 *WSGI* 服务，如果只是简单使用的话，与 *django* 内置的调试服务无区别。

首先是在 python 虚拟环境安装

```shell
$ sudo pip install gunicorn
```

进入项目主目录，运行

```shell
# 执行 your_project 目录下的 wsgi.py
$ sudo gunicorn your_project.wsgi

[2017-08-31 14:19:48 +0000] [28801] [INFO] Starting gunicorn 19.7.1
[2017-08-31 14:19:48 +0000] [28801] [INFO] Listening at: http://127.0.0.1:8000 (28801)
[2017-08-31 14:19:48 +0000] [28801] [INFO] Using worker: sync
[2017-08-31 14:19:48 +0000] [28806] [INFO] Booting worker with pid: 28806
```

默认使用 *8000* 端口，是不是很简单呢？纯粹测试时很简单，但真正部署到服务器上，还有一段要走呢！

先来看看 [*Gunicorn* 的使用参数](http://docs.gunicorn.org/en/latest/run.html)，讲主要的几个

- `-c CONFIG, --config=CONFIG` ：配置文件，可以将使用的参数写在配置文件中，执行时调用即可。
- `-b BIND, --bind=BIND`：指定服务的 *socket* 来绑定，可以是主机名或IP，或者是IP：端口，以及 *.sock* 文件
- `-w WORKDERS, --workers=WORKERS` ：*worker* 进程的个数，通常是每个CPU核心分配 `2~4` 个 *workers* 。
- `-k WORKDERCLASS, --worker-class=WORKDERCLASS`： *worker* 进程的类型，有`sync` 、`eventlet`、`gevent`、`tornado`、`gthread`、 `gaiohttp`这几种类型，默认的是 `sync` 。

更详细的 Gunicorn参数 请参见 [Settings](http://docs.gunicorn.org/en/latest/settings.html#settings) 。

来看几个运行例子

```shell
# 进入项目主目录
$ cd your_project_root_directory

# 更改绑定的端口
$ gunicorn your_project.wsgi:application -b 127.0.0.1:9999

# 不使用端口而使用 socket 文件
$ gunicorn your_project.wsgi:application -b unix:/tmp/test.sock

# 使用两个 worker
$ gunicorn your_project.wsgi:application -w 2
```

好了，测试运行正常，那么问题来了，如果要重启时，怎么弄？手动 *kill* 掉，再运行一次长长的命令？以及如何监听 *gunicorn* 的运行状态呢？ *gunicorn* 官方推荐了几种 [monitoring方式](http://docs.gunicorn.org/en/latest/deploy.html#monitoring)

### [*Supervisor*](http://supervisord.org/running.html) 

新建一个 `gunicorn.conf` 配置

```shell
# 进入 supervisor 自定义配置目录
$ cd /etc/supervisor/conf.d

# 新建配置
$ sudo cat > gunicorn.conf << EOF
[program:gunicorn] # 配置的程序名字
# 实际的gunicorn命令执行，调用指定的配置文件
command="python虚拟环境中gunicorn的绝对路径" your_project.wsgi:application -c gunicorn_args.conf.py
directory="项目目录"
user=your_user # 以什么用户运行
autostart=true
autorestart=true
redirect_stderr=true # 重定向到标准输出
stdout_logfile="日志文件路径"
EOF
```

接着是 *gunicorn* 参数的配置文件 `gunicorn_args.conf.py`

```python
import multiprocessing

bind = "unix:/.../.../***.sock"   # 绑定 socket
workders = 2    # workder数量
loglevel = "error"  # 日志级别:错误级别
logfile = "-"  # - 表示日志定向到标准输出
name = "your_project"  # 进程名字，影响在 ps 或 top 命令时的进程名显示
user = "your_user"  # 以什么用户名执行
group = "your_user_group"  # 以什么群组执行
```

手动执行命令看是否有报错

```shell
$ gunicorn your_project.wsgi:application -c gunicorn_args.conf.py
```

没问题则启动  *supervisor* 来管理

```shell
# 使用 supervisord 方式
sudo /etc/init.d/supervisord restart

# 使用 supervisorctl 方式
sudo supervisorctl reread
sudo supervisorctl restart

sudo supervisorctl [stop|status|restart]
```

接着设置 *nginx* ，使用代理的方式，让请求转到 gunicorn 中去处理

新建一个 nginx 配置： `gunicorn.conf`

```shell
$ sudo cd /etc/nginx/site-available/

$ sudo cat > gunicorn.conf << EOF
# 定义实际处理的请求
upstream app_server {
  server unix:/.../.../***.sock fail_timeout=0;  # gunicorn 中定义的socket
}

# 如无主机匹配，关闭连接以阻止跳转
server {
  listen 80 default_server;
  return 444;
}

server {
  listen 80;
  client_max_body_size 4G;
  
  server_name example.com
  
  keepalive_timeout 5;
  
  # 静态文件目录
  root /path/to/app/current/public;
  
  # 检测静态文件，如无则使用代理跳转到应用
  location / {
    try_files $uri @proxy_to_app;
  }
  
  # 实际的代理设置
  location @proxy_to_app {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    # proxy_set_header X-Forwarded-Proto https;  # 使用HTTPS时才开启
    # proxy_buffering off; # 使用长轮询，websocket则需要关闭此项
    proxy_set_header Host $http_host;
    proxy_redirect off; # 有上述host的定义则不需要 重定向 了
    proxy_pass http://app_server;
  }
  
  error_page 500 502 503 504 /500.html;
  location = /500.html {
    root /path/to/app/current/public;
  }
}
EOF
```

启用之并访问浏览器进行测试

```shell
# 启用配置
$ sudo ln -s /etc/nginx/site-available/gunicorn.conf /etc/nginx/site-enabled/gunicorn

# 重启nginx
$ sudo /etc/init.d/nginx restart

# 访问
$ curl http://your_website
```

### [Systemd](http://docs.gunicorn.org/en/latest/deploy.html#systemd)

创建 *gunicorn* 服务：`/lib/systemd/system/gunicorn.service` 

```shell
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
PIDFile=/run/gunicorn/pid
User=jeff
Group=jeff
RuntimeDirectory=gunicorn
WorkingDirectory=/data/py_pro/py2.7/website
ExecStart=/data/py_pro/py2.7/bin/gunicorn --pid /run/gunicorn/pid --bind unix:/run/gunicorn/socket website.wsgi --workers 3 --log-level error --log-file /dat
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s TERM $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target

```

创建对应的 *socket* 服务：`/lib/systemd/system/gunicorn.socket` 

```shell
[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/run/gunicorn/socket

[Install]
WantedBy=sockets.target
```

> 注意：gunicorn.service 与 gunicorn.socket 必须前缀名相同，否则启动失败

创建临时目录配置：`/etc/tmpfiles.d/gunicorn.conf` 

```shell
# 临时文件类型  路径  权限  用户名  群组名
d /run/gunicorn 0755 someuser somegroup -
```

设定 *socket* 开机启动

```shell
systemctl enable gunicorn.socket
```

开启 *socket* 服务

```shell
systemctl start gunicorn.socket
```

测试下 *socket* 是否正常运用

```shell
curl --unix-socket /run/gunicorn/socket http
```

修改 *nginx* 配置中的 *socket* 路径

```shell
# new socket setting in nginx configuration
unix:/run/gunicorn/socket
```

OK ，到此设置完毕！

## uWSGI 与 Nginx

*uWSGI* 是一种快速，自处理并且面向开发者和系统管理员的应用容器服务，使用C语言开发的。其用法与 *Gunicorn* 类似，语法不同而已。

首先仍然是安装，在虚拟环境中

```shell
$ sudo pip install uwsgi
```

运行 django 项目

```shell
# 进入项目主目录
$ sudo uwsgi --http :8000 --module your_project.wsgi
```

*django* 官方文档中的启动示例

```shell
uwsgi --chdir=/path/to/your/project \   # 切换到项目主目录
    --module=your_project.wsgi:application \  # 调用 wsgi
    --env DJANGO_SETTINGS_MODULE=your_project.settings \
    --master --pidfile=/tmp/project-master.pid \
    --socket=127.0.0.1:49152 \      # 也可以是一个文件
    --processes=5 \                 # worker进程数量
    --uid=1000 --gid=2000 \         # 指定用户与群组，慎用root
    --harakiri=20 \                 # 超过20秒后重启进程
    --max-requests=5000 \           # 超过5000个请求时重启进程
    --vacuum \                      # 退出时清理环境变量
    --home=/path/to/virtual/env \   # virtualenv目录
    --daemonize=/var/log/uwsgi/yourproject.log      # 后台运行进程，日志文件
```

通过浏览器访问项目主页，查看是否正常，此时的运作流程为：

```shell
the web client <-> uWSGI <-> Django
```

接着加入 *nginx* ，新建一个配置： `uwsgi.conf`

```shell
$ sudo cat > /etc/nginx/site-available/uwsgi.conf << EOF
upstream uwsgi_django {
  # server 127.0.0.1:8001; 使用端口
  # 使用sock文件比端口能减少更多的系统开支，速度更快；免去端口分配之苦
  server unix://path/to/your/project/project.sock; 
}

server {
  listen 80;
  server_name example.com;
  charset utf-8;
  
  client_max_body_size 75m;
  error_log /tmp/your_define_error_name.log; # 开启错误日志方便调试
  
  # media目录
  location /media {
    alias /path/to/your/project/media;
  }
  
  # 静态资源目录
  location /static {
    alias /path/to/your/project/static;
  }
  
  # 把除上述所有的请求转发到 uwsgi_django 处理
  location / {
    uwsgi_pass  uwsgi_django;
    include /path/to/your/project/uwsgi_params;
  }
}
EOF
```

如果 `unix://path/to/your/project/project.sock` 文件无法连接，并报权限错误时

```shell
connect() to unix:///path/to/your/mysite/mysite.sock failed (13: Permission
denied)
```

可在 `uwsgi` 执行时指定权限

```shell
uwsgi --socket mysite.sock --wsgi-file test.py --chmod-socket=666 # (very permissive)
```

另外， `uwsgi_params` 的设置可参考 [nginx uwsgi_params settings](https://github.com/nginx/nginx/blob/master/conf/uwsgi_params) ，其详细内容为

```shell
uwsgi_param  QUERY_STRING       $query_string;
uwsgi_param  REQUEST_METHOD     $request_method;
uwsgi_param  CONTENT_TYPE       $content_type;
uwsgi_param  CONTENT_LENGTH     $content_length;

uwsgi_param  REQUEST_URI        $request_uri;
uwsgi_param  PATH_INFO          $document_uri;
uwsgi_param  DOCUMENT_ROOT      $document_root;
uwsgi_param  SERVER_PROTOCOL    $server_protocol;
uwsgi_param  REQUEST_SCHEME     $scheme;
uwsgi_param  HTTPS              $https if_not_empty;

uwsgi_param  REMOTE_ADDR        $remote_addr;
uwsgi_param  REMOTE_PORT        $remote_port;
uwsgi_param  SERVER_PORT        $server_port;
uwsgi_param  SERVER_NAME        $server_name;
```

最后再处理 `uwsgi` ，嘿嘿！

这回把 *uwsgi* 的[启动参数](https://uwsgi.readthedocs.io/en/latest/Configuration.html)写到配置文件中去，配置文件的格式有多种，如`ini` ，`json` ， `yaml` ，`xml` ，此处使用`yaml` 。把官方示例的启动参数改成 `your_project_uwsgi.yaml` 

```yaml
# your_project_uwsgi.yaml

uwsgi:
	# django 设置
	home: /path/to/virtualenv
	chdir: /path/to/your/project
	module: project.wsgi
	env: DJANGO_SETTINGS_MODULE=project.settings
	daemonize: /var/log/uwsgi/yourproject.log
	
	# 进程相关的设置
	pidfile: /tmp/project-maser.pid
	socket: /tmp/uwsgi.sock  # 测试单条命令时使用 http-socket: 0.0.0.0:9000
	master: 1
	workers: 3
	processes: 5
	uid: 1000
	gid: 1000
	harakiri: 20  # 调试是否正常运行时，禁用此项
	max-requests: 5000  # 调试是否正常运行时，禁用此项
	vacuum: 1
	chmod-socket: 666
```

运行测试下

```shell
$ uwsgi --yaml your_project_uwsgi.yamle
```

OK，现在问题来了，如何对 *uwsgi* 进行类似 重启、关闭的操作呢？

重启

```shell
# 方法一
$ kill -HUP $(cat /tmp/project_master.pid)

# 方法二
$ uwsgi --reload /tmp/project_master.pid
```

关闭

```shell
$ kill -INT $(cat /tmp/project_master.pid)

uwsgi --stop /tmp/project_master.pid
```

还可以借助  [Systemd](https://uwsgi-docs.readthedocs.io/en/latest/Systemd.html) 来管理，此处不展开了。

到此，再重启 *nginx* 令新配置生效，可以进行测试了。

如果在线上服务器部署时，可以这样设置

```shell
# 在全局系统环境中安装 uwsgi
$ /usr/local/bin/pip install uwsgi

# 启用文件监控模式 emperor，但凡文件有变动，会重启服务
$ uwsgi --emperor --yaml your_project_uwsgi.yamle

# 添加到开机启动
$ /usr/local/bin/uwsgi --emperor --yaml your_project_uwsgi.yamle
```

此时，浏览项目主页的运作流程则变为

```shell
the web client <-> the web server <-> the socket <-> uWSGI <-> Python
```

## 小结

撒花……终于到小结了……嘿嘿！！！

看完整篇文章，对这3种部署方法应该知道如何取舍了吧？

| 组合类型              | 优势                | 劣势                                    |
| ----------------- | ----------------- | ------------------------------------- |
| uWSGI 与 Nginx     | 性能表现最优            | 配置复杂，上手较难，uWSGI文档较难阅读                 |
| Gunicorn 与 Nginx  | 纯python写的，更少的内存占用 | 额……好像操作没apache简便而已，文档比 uWSGI 好        |
| Apache 与 mod_wsgi | 操作简便              | 处理高并发较差，部分配置内容复杂，有两个主要版本，window的部署最蛋疼 |

个人使用的方案是  `Gunicorn Nginx` ，对性能有要求，首选第一种咯。

只想使用 `Apache Mod_wsgi ` 的组合，又想利用 *nginx* 的高并发的优势？可以使用 *nginx* 做前端代理，处理客户端的请求，只利用其处理静态资源文件的高性能 ，其他请求则代理转发到 *apache* 后端处理，此法存在，但不是主流，具体设置另外发文再写了。

以上部署过程，始终明确两点：

1. 要明白当你在浏览器访问项目主页时，其背后的运作流程是什么？
2. `目录规划` ，特别是 `日志目录` ，`.sock` 文件的路径处理「最好不放在/tmp」

好了，可以收摊了……



## 参考链接

> [How to deploy with wsgi](https://docs.djangoproject.com/en/1.10/howto/deployment/wsgi/)
>
> [how to use django with gunicorn](https://docs.djangoproject.com/en/1.10/howto/deployment/wsgi/gunicorn/)
>
> [how to use django with uWSGI](https://docs.djangoproject.com/en/1.10/howto/deployment/wsgi/uwsgi/)
>
> [how to use django with apache and mod_wsgi](https://docs.djangoproject.com/en/1.10/howto/deployment/wsgi/modwsgi/)
>
> [gunicorn deployment with nginx](http://docs.gunicorn.org/en/latest/deploy.html)
>
> [setting up django and your web server with uWSGI and nginx](https://uwsgi-docs.readthedocs.io/en/latest/tutorials/Django_and_nginx.html)
>

