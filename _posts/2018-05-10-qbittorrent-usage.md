---
layout: post
title: qBittorrent下载の道
date: 2018-05-10
tags: [skill]
feature-img: "https://images.pexels.com/photos/436781/pexels-photo-436781.jpeg?cs=srgb&dl=access-accessories-blur-436781.jpg&fm=jpg"
---



弱水三千，只取一瓢。众多BT软件，只取 qBittorrent，「`以下简称QB`」！


<!--more-->


## 扯淡

遥想当年，BT软件大行其道，电视剧、电影、游戏、软件，但凡能BT下载的，都不放过。迅雷的火爆，让蛋疼的我们竟然无聊得比等级高低，看谁下载的多。`松鼠症` 也是在这个时间节点患上的。
随着视频网站的兴起，大家观影的习惯都随之改变，如今的迅雷有点中年不举的态势，`下载` 这个东东终究会被时间所淘汰吗？不存在的！！！精确的说，在现行的环境下，`下载` 成了一条分隔线，把对观影有要求的用户筛选出来了。

## 安装

官网在 [这里](https://www.qbittorrent.org/)，下载安装即可。Ubuntu的用户得添加PPA源，否则安装的是旧版。

好了，安装完毕，可以散场了??……

## 搜索

使用下载软件的第一问题是：`从哪来？`。资源呢？通常有两种方式

- 逆向搜索

看影片，得找字幕吗？到字幕站去找片名，例如 

```bash
Westworld.S02E02.reunion.1080p.WEB.H264-DEFLATE.mkv
```

找到片名后，贴到搜索引擎，或者是BT软件中即可。

`QB` 自带BT搜索引擎，需要手动开启，在 `视图` => `搜索引擎` 选中即可。默认自带了几个搜索引擎，支持安装额外的 `搜索插件` ， 可在 [这里](https://github.com/qbittorrent/search-plugins/wiki/Unofficial-search-plugins) 查找。部分插件带有 `不可描述` 的属性??……

搜索不要求自备梯子，一站式的用户体验，啧啧。

- 正向搜索

同样是找片名，`正向搜索` 是先确定影片的英文名，例如想看 「黑豹」，可在 [豆瓣](http://www.douban.com) 查询英文名为 *Black Panther* ， 然后在你所知道的网站中搜索。

我哪知道有什么BT网站呢？同样是从影片名入手，例如

```bash
Black.Panther.2018.1080p.BluRay.H264.AAC-RARBG.mkv
```

最后面的 **RARBG** 即为BT网站的名字！关于影片名字的解释，可参考 [这里](https://www.jianshu.com/p/6c6e1cd1d4aa)。

找到相应的种子 或者 磁力链接时，再使用 `QB` 下载即可。


## RSS订阅

总不可能经常有事没事逛下载站吧？这对于 `松鼠症` 的朋友来说，根本不是问题啊??。

`QB` 支持上古神器：`RSS` 订阅，例如这个订阅源， [RARBG-MOVIE](https://rarbg.to/rssdd.php?categories=14;15;16;17;21;22;42;44;45;46;47;48) 。其他资源网站也会提供 RSS 订阅地址的，自行添加即可。如此一来，既可跳过烦人的广告，又能快速定位想下载的影片，快哉?。


## 命令行

QB 支持命令行模式运行，不需要图形界面，但可以通过远程访问 web页面 ，详细的说明请参考 [这里](https://github.com/qbittorrent/qBittorrent/wiki/Running-qBittorrent-without-X-server)。 

```bash
# 安装
$ sudo apt install qbittorrent-nox

# 运行 更改端口添加参数 --webui-port=端口号
$ qbittorrent-nox
*** Legal Notice ***
qBittorrent is a file sharing program. When you run a torrent, its data will be made available to others by means of upload. Any content you share is your sole responsibility.

No further notices will be issued.

Press 'y' key to accept and continue...
y

******** Information ********
To control qBittorrent, access the Web UI at http://localhost:8080
The Web UI administrator user name is: admin
The Web UI administrator password is still the default one: adminadmin
This is a security risk, please consider changing your password from program preferences.
```

浏览器访问 `http://x.x.x.x:8080` ，输入账号密码就可以使用了。实际使用时，最好拿 *nginx* 做反代。

## 视频封装

> 顺带把 `视频封装` 这个给补上，字幕校对时间轴的不想写了……

`mkvtool` 的官网在 [这里](https://mkvtoolnix.download/)，一个非常强大的视频封装软件。

通常下载整季美剧时，需要手动配上字幕，鉴于不喜欢视频、字幕分离，于是全都封装成 MKV ，逐个合成会死人的，使用命令行方式能够快速完成。

举个栗子，去除源文件所有字幕，额外添加两个字幕，并指定默认字幕为简体中文，则有如下

```bash
$ mkvmerge -o 输出名字.mkv -S 源视频.mkv \
  --language 0:chi --track-name 0:简体中文 简体中文.srt \
  --default-track 0 --language 0:eng \
  --track-name 0:官方英文 英文字幕.srt
```

对 `mkvmerge` 参数有兴趣的，可参考 [这里](https://mkvtoolnix.download/doc/mkvmerge.html)。

整季美剧封装时，在 `shell` 里写个 `for循环` 执行就可以了。

某些国产液晶电视，直接怼U盘进去，只能播放 Mp4 格式的视频，封装成 MKV 后估计不行，未测试。


## 小结

闲聊到此，此文观点不代表本人支持盗版?！！！

