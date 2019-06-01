---
title: CentOS布署Django+Apache
layout: post
date: 2016-10-21 00:00:00
tags: [Django, Linux]
feature-img: "assets/img/pexels/startrails.jpg"
---



工作需要，必须得在 *CentOS* 中布署 *Django* ，并且使用 *apache* ，以 `mod_wsgi`串连着使用。如果是使用 *Nginx* ，还得更加麻烦，使用的则是 *uwsgi* 。之前写的这个教程，仅仅是成功安装 *django* ，并且能够使用内置的 **开发服务器** 来调试使用。现在无非就是借助*Apache* 嘛，应该也很简单吧……果然老子还是太年轻了……天朝内现有的教程文章，要么过于简(chao)洁(xi)，完全没详细的原理讲解；要么是版本太旧，根本无法使用，哎……

<!--more-->


最后，历经一个下午的折腾，终于让我成功解决问题了，难点在于 

1. Python 与 Mod_wsgi的关联
2. Apache识别到django项目虚拟环境中的wsgi.py

好了， 别的不扯，上干货。

## 版本信息

- CentOS 6.8
- Apache 2.2.15
- mod_wsgi 4.5.6
- django 1.10
- Python 2.7.12

## 安装Python

*CentOS* 默认安装的 **python** 版本是 *2.6.6* 的，使用 *django* 则必须升级到 *2.7.xx* 版本。另外，*CentOS* 默认未安装 **sqlite3** ，因此编译安装升级 *python* 时，需要添加对其的支持。

```shell
# 安装 sqlite3
yum -y install sqlite-devel

# 下载安装python2.7.12
cd /usr/src
wget https://www.python.org/ftp/python/2.7.12/Python-2.7.12.tgz
tar -xf Python-2.7.12.tgz
cd Python-2.7.12
./configure --prefix=/usr/local --enable-shared
make all
make install

# 替换原换文件，并解决Yum依赖2.6.6的问题
mv /usr/bin/python /usr/bin/python2.6.6
ln -s /usr/local/bin/python2.7 /usr/bin/python
sed -i 's/python/python2\.6\.6/g' /usr/bin/yum

# easy_install 必须先安装它，再安装pip
cd /usr/src
wget https://pypi.python.org/packages/b5/9a/9ce1a45a076f977cb870bf0c937\
0347c9371b0e9aa9ca9859196ce58afda/setuptools-28.6.1.tar.gz#\
md5=b8df391e7532b544e16e1e4cc35a90e5
tar -xf setuptools-28.6.1.tar.gz
cd setuptools-28.6.1
python setup.py install

# pip
cd /usr/src
wget https://pypi.python.org/packages/e7/a8/7556133689add8d1a54c0b14a\
eff0acb03c64707ce100ecd53934da1aa13/pip-8.1.2.tar.gz#\
md5=87083c0b9867963b29f7aba3613e8f4a
tar -xf pip-8.1.2.tar.gz
cd pip-8.1.2
python setup.py install

# bpython，这个力荐安装
pip install bpython
# 如果pip安装失败，则
easy_install bpython
```

`警告`： 注意 *python* configure的选项

```shell
./configure --prefix=/usr/local --enable-shared
```

后边的*--enable-shared* 非常重要，如果缺少的这个选项，则无法将 *python* 与 *apache* 进行连接。

升级安装*python*后，执行`python -V` 报如下错的话：

*python: error while loading shared  libraries: libpython2.7.so.1.0: cannot open shared object file: No such file or directory*

则需要做如下操作

```shell
echo '/usr/local/lib' > /etc/ld.so.conf.d/python2.7.conf
ldconfig
python -V
```


## 安装django
以源码的方式安装 *django* ，截止目前最新的版本是 *1.10.1* 。再者是平时均在虚拟环境下使用，因此把*virtualenv* 也安装上。

```shell
# 安装django
wget https://www.djangoproject.com/m/releases/1.10/Django-1.10.1.tar.gz
tar -xf Django-1.10.1.tar.gz
cd Django-1.10.1
python setup.py install 

# 安装virtualenv
pip install virtualenv
```

关于*virtualenv* 的使用比较简单，结合实操来顺带讲解吧

```shell
# 新建django项目
cd /data
# 新建虚拟环境
virtualenv mysite
# 切换到虚拟环境
cd /data/mysite
source bin/activate
# 检查效果
which python
# 布署django项目
django-admin startproject web
# 退出虚拟环境
deactivate
```

假设我的 *django* 项目布署在上述目录中，一会 *apache* 中的配置以上述路径为准。

## Apache2.2 与 mod_wsgi

最心酸的部分来了……

起初网上胡乱一通找教程，*mod_wsgi* 不想以源码方式安装，想着直接用 *yum* 比较省事。看来 **ubuntu** 还是比较适合我啊，用得顺手多了，哎……

```shell
# 网上找的教程yum安装 Mod_wsgi
yum install epel-release
yum install mod-wsgi

# 安装完毕后，/etc/http/conf.d/会多出个文件mod_wsgi.conf
# 内容为；apache会自动加载
LoadModule  wsgi_module modules/mod_wsgi.so
```

好了，*apache* 的插件解决了，接下来就是新建 apache 的虚拟主机配置了，也挺简单的嘛。照搬*ubuntu* 上的配置文件，哎哟喂，出问题了……

```shell
# apache 2.2 与 2.4的区别
## 2.2
Order deny, allow
Allow from all

## 2.4
Require all granted

# apache虚拟主机中配置django的虚拟环境
## ubuntu 16.04 + apache 2.4 中的虚拟主机配置
<VirtualHost *:80>
LogLevel debug
WSGIScriptAlias / /data/mysite/web/web/wsgi.py
WSGIDaemonProcess mysite python-path=/data/mysite/web:/data/mysite/lib/python2.7/site-packages
WSGIProcessGroup mysite
</VirtualHost>

## CentOS 6.8 + apache 2.2 中的虚拟主机配置
WSGIPythonPath /data/wtweb/:/data/wtweb/lib/python2.7/site-packages
<VirtualHost *:80>
WSGIScriptAlias / /data/wtweb/web/web/wsgi.py
...
</VirtualHost>
```

特别注意下*WSGI* 的写法，有名字上的区别噢，并且下面的*WSGI* 配置不能写在*VirtualHost* 里头。个人认为造成异同应该是apache版本与 系统版本。解决了这小段弯路，再来看看重量级别的。

我打开apache的日志记录，一边调试一边尝试解决各种报错问题。配置文件中记得开启日志级别为`debug` ，方便排查问题。明明已经安装了`mod_wsgi`啊，怎么就出错呢？

此时的我已经非常、极度烦躁了……久久未能有结果……大谷歌上也没有解答……烦死了……继续查看日志记录……有这么一条……

*[notice] Apache/2.2.15 (Unix) DAV/2 mod_wsgi/3.2 Python/2.6.6 configured*

我擦，大爷的，老子的*python* 不是已经升级到了*2.7.12*了吗？怎么还显示 *2.6.6* 呢。此时此刻，老夫平复了自己烦躁的状态，思考了一番，这肯定是*mod_wsgi*有问题，Yum安装下来的它，默认应该是关联到*CentOS*默认安装的*python 2.6* ，而我现在以源码方式升级了Python。那么问题来了，这尼玛铁定是要老子以源码的方式安装*mod_wsgi*啦，这就是解决`mod_wsgi与python版本关联` 的问题了。

```shell
# 卸载yum安装的Mod_wsgi
yum remove mod_wsgi

# apache 的apxs扩展
yum install httpd_devel

# 下载源码最新版，估计要墙
cd /usr/src
wget https://codeload.github.com/GrahamDumpleton/mod_wsgi/tar.gz/4.5.6
tar -xf mod_wsgi-4.5.6.tar.gz
cd mod_wsgi-4.5.6
# 关联编译升级安装的python2.7
./configure --with-python=/usr/local/bin/python2.7
make && make install
```

好了，这会再看看*apache*日志记录，嘿嘿……

*[notice] Apache/2.2.15 (Unix) DAV/2 mod_wsgi/4.5.6 Python/2.7.12 configured*

接下来，则是解决 `Apache如何识别django项目虚拟环境中的wsgi.py` 了。解决方式如下：

```shell
# 在django项目的wsgi.py文件中添加这两句，让系统能够找到实际的django项目路径
# 这样才能找到项目的配置文件，此处为我项目的 web/web/setting.py
import sys
sys.path.append(r'/data/mysite/web')
```

好了，问题终于完美解决了……最后上个完整版的*apahce* 虚拟主机配置文件

```shell
cat > /etc/httpd/conf.d/mysite.conf << EOF
WSGIPythonPath /data/mysite/:/data/mysite/lib/python2.7/site-packages
<VirtualHost *:80>
    ServerName mysite.test.com
    ServerAdmin root@local.com
    DocumentRoot /data/mysite/web
    LogLevel debug
  
    Alias /media/ /data/mysite/web/media/
    Alias /static/ /data/mysite/web/collected_static/

    # WSGIScriptAlias 一定要放在virtualhost内，不然默认主页无法打开
    WSGIScriptAlias / /data/mysite/web/web/wsgi.py
    <Directory "/data/mysite/web/web">
        <Files wsgi.py>
        Order deny,allow
        Allow from all
        </Files>
    </Directory>
  
</VirtualHost>
EOF
```

## 小结

最后的小结部分，谈谈感受吧。网络上有的教程皆是陈旧，不适用的；谷歌上也没有类似的情况做解答。上述的成果，耗费了我将近6个小时有多。越是找不到解决方法，而耗费了这么多时间，心里越是着急。兴许是最后的灵光乍现，让我从*apache* 的日志记录中发现了端倪，解决了最为关键的部分。思路与心态混乱之时，应该平静下来，重新梳理一遍流程，查找关键点，了解自身的环境状况，网络上的教程千差万别，不可盲目跟随。哎……

## 参考链接

> [Python编译安装使用开启共享](http://blog.sina.com.cn/s/blog_6b1c9ed50101aig5.html)
>
> [virtualenv使用官方简要](http://docs.python-guide.org/en/latest/dev/virtualenvs/)
