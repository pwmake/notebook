---
layout: post
title: youtube-dl使用示例教程
date: 2017-08-11
tags: [Skill]
feature-img: "https://images.pexels.com/photos/34407/pexels-photo.jpg?cs=srgb&dl=app-apple-hand-34407.jpg&fm=jpg"
---


为了下载一个 `Super Simple Songs` 的播放列表 ，花了点时间来研究 `youtube-dl` 这个神器，唯下载资源时，方恨网速太慢啊，哎……

*youtube* 比天朝内的视频网站良心太多了，片头广告播放5秒后可选择跳过，中途有广告，亦可选择跳过，更重要的是还能观看某些「河蟹」内容。

<!--more-->

## 安装 [ffmpeg](https://ffmpeg.org/ffmpeg.html)

ffmpeg是首要基础，`youtube-dl` 下载完成后会自动调用，对视频进行封装。以下是 *Ubuntu 16.04* 的安装

```bash
# 添加软件源
$ sudo add-apt-repository ppa:jonathonf/ffmpeg-3

# 更新并安装
$ sudo apt update && sudo apt install ffmpeg libav-tools x264 x265

# 卸载官方源的2.8版本
$ sudo apt autoremove
```

## 安装 [youtube-dl](https://github.com/rg3/youtube-dl/blob/master/README.md#readme)

各种方式五花八门，瘟系统 用户要下载个 *.exe* 

```bash
# curl安装
$ sudo curl -L https://yt-dl.org/downloads/latest/youtube-dl -o /usr/local/bin/youtube-dl
$ sudo chmod a+rx /usr/local/bin/youtube-dl

# wget安装
$ sudo wget https://yt-dl.org/downloads/latest/youtube-dl -O /usr/local/bin/youtube-dl
$ sudo chmod a+rx /usr/local/bin/youtube-dl

# pip安装
$ sudo -H pip install --upgrade youtube-dl

# brew安装
$ brew install youtube-dl
```

## 基本用法

官方文档写得非常详细，此处以大多数人使用的逻辑来讲解。此外，一个 *youtube* 上的视频文件，包含 `视频` 和 `音频` 两样，不同质量的它们可以有多种组合。

假设下载 「Super Simple Song」里的 [If you're happy](https://www.youtube.com/watch?v=l4WNrvVjiTw) 。

- 下载最优视频

```bash
$ youtube-dl "https://www.youtube.com/watch?v=l4WNrvVjiTw"
```

不带任何参数，则默认下载画质、音质最好的文案。

- 查看可选的视、音频格式

```bash
$ youtube-dl -F "video_url"
```

得出以下信息

```bash
139          m4a        audio only DASH audio   51k , m4a_dash container, mp4a.40.5@ 48k (22050Hz), 18.65MiB
249          webm       audio only DASH audio   76k , opus @ 50k, 22.62MiB
250          webm       audio only DASH audio   97k , opus @ 70k, 29.38MiB
140          m4a        audio only DASH audio  131k , m4a_dash container, mp4a.40.2@128k (44100Hz), 49.80MiB
171          webm       audio only DASH audio  161k , vorbis@128k, 48.73MiB
251          webm       audio only DASH audio  176k , opus @160k, 56.33MiB
160          mp4        256x144    DASH video  111k , avc1.4d400c, 30fps, video only, 20.48MiB
278          webm       256x144    144p  122k , webm container, vp9, 30fps, video only, 31.68MiB
242          webm       426x240    240p  265k , vp9, 30fps, video only, 51.28MiB
133          mp4        426x240    DASH video  271k , avc1.4d4015, 30fps, video only, 33.36MiB
243          webm       640x360    360p  470k , vp9, 30fps, video only, 89.86MiB
134          mp4        640x360    DASH video  604k , avc1.4d401e, 30fps, video only, 64.25MiB
244          webm       854x480    480p  859k , vp9, 30fps, video only, 141.71MiB
135          mp4        854x480    DASH video 1192k , avc1.4d401f, 30fps, video only, 116.73MiB
247          webm       1280x720   720p 1678k , vp9, 30fps, video only, 270.21MiB
136          mp4        1280x720   DASH video 2082k , avc1.4d401f, 30fps, video only, 180.51MiB
248          webm       1920x1080  1080p 2923k , vp9, 30fps, video only, 522.59MiB
137          mp4        1920x1080  DASH video 3087k , avc1.640028, 30fps, video only, 268.58MiB
17           3gp        176x144    small , mp4v.20.3, mp4a.40.2@ 24k
36           3gp        320x180    small , mp4v.20.3, mp4a.40.2
43           webm       640x360    medium , vp8.0, vorbis@128k
18           mp4        640x360    medium , avc1.42001E, mp4a.40.2@ 96k
22           mp4        1280x720   hd720 , avc1.64001F, mp4a.40.2@192k (best)
```

- 自行组合方案

从上述的信息中，按需挑选自己想要的视频、音频组合方案，填写对应的序号

```bash
# 1080P的mp4视频 与 44100Hz的音频
$ youtube-dl -f '137+140' "video_url"
```

是不是很简单呢！

- 字幕

如果视频带有字幕的话，则一并下载

```bash
# 下载字幕，并按顺序选择 ass/srt/best 字幕，把字幕转成 srt 格式
$ youtube-dl --write-sub --sub-format "ass/srt/best" --convert-subs "srt" "video_url"
```

`—write-sub` ：写入字幕，即把字幕下载。

`--sub-format`：指定字幕格式，按顺序选，不存在则选下一个。

`--convert-subs`： 转换字幕，格式有限制，通用为 *srt* ；若不转，某些字幕可能是 *.vtt* 的；如果有 *ass* 字幕可下载，则无须加此项。 

- 代理下载

若借助局域网内的 `SS` 来下载，则

```bash
# 自行填写IP和端口
$ youtube-dl --proxy "socks5://x.x.x.x:port" "video_url"
```

- 其他

买东西时能光看不买，同理，可以光查看下载信息而不下载吗？能！

```bash
--skip-download # 跳过下载
-g  # 只打印下载链接
-e  # 只打印标题
```

还有一个重点的模块是 `定制输出模板` ，即视频下载好后，`ffmpeg` 该如何按照你想要的格式、信息来输出视频文字。此处不展开。

以上需求能满足绝大多数人了，嘿嘿！

## 下载播放列表

是时候祭出这个大杀器「心酸只有自己才知道」了……

下载这个 [Super Simple Song](https://www.youtube.com/watch?v=kz_EQSfFx0g&index=19&list=RD8ljHzljQrY8) 的播放列表，总共有 50 个文件。

直接在本地通过代理下载时，对 VPS 和 本地网络 的要求比较高，我多是直接在 *VPS* 下载好，再拖回到本地。

- 壕下载法

此法最为简单粗暴，直接 `youtube-dl "video_url"` 即可，缺点是总视频大小超过14G，VPS硬盘空间小的慎用。

- 穷人下载法

何为穷？直接选最差的组合唄。

```bash
$ youtube-dl -f worstvideo "video_url"
```

- 折中选择法-01

选好指定的格式，如 视频选 *135* ，音频选 *139* 

```bash
$ youtube-dl -f '135+139' "video_url"
```

此法缺点，有一定机率爆……哎哟，怎么报错了？没有指定的格式编号可选？列表下载到一半停止了，怎么整？

从哪里报错，从哪里继续。报错信息会显示是第几个视频，记下该编号，假设是 51 个视频当中的第 9 个出错了，先跳过它，则

```bash
# --playlist-start 列表开始编号，默认是1
# --playlist-end 列表结束编号，默认是最大值

# 从列表的第10个视频开始下载到最后
$ youtube-dl -f '135+139' --playlist-start 10 
```

- 折中选择法-02

方法01能用，但有一定机率需要人工干预，不能保证自动化。则可以

指定多个格式编号做备选，按顺序做选择，不存在则自动选下一个，注意是 `视频, 音频` ， `,` 表示与，`/` 表示或

```bash
$ youtube-dl -f '136/135/134,140/139' "video_url
```

再下载试试看？嗯……嗯哼？怎么部分简短的视频文件这么小？能否实现是简短视频时，则下载它们的最优组合呢？

当然可以咯。只需要对上述规则再过滤一下即可。

```bash
# 设定一个文件大小的标准，例如视频文件小于15M时，选最优；音频小于1M时，选最优
$ youtube-dl -f '135[filesize>=15M]/bestvideo,139[filesize>=1M]/bestaudio' "video_url
```

过滤规则除了文件大小，还可以是视频的 `宽「width」` 和 `高「height」` 

例如只选 *720P* 的视频时，则为 `-f '135[height=720]'` ，注意过滤规则注意写在 *中括号* 内。

## 脚本调用

在 *python* 脚本中调用时，可以写成一个供调用的函数: `download.py`

```python
#! /usr/bin/python3

from os import rename
import youtube_dl


def download(youtube_url):
    """
    download the video from the given url
    """
    
    def rename_hook(d):
        """
        youtube-dl's hook to rename the downloaded video name
        """
        
        if d['status'] == 'finished':
            file_name = '你想要的新文件名'
            rename(d['filename'], file_name)
    
    # 定义某些下载参数        
    ydl_opts = {
        # 'format': 'bestaudio/best',
        'progress_hooks': [rename_hook],
        # outtmpl 格式化下载后的文件名，避免默认文件名太长无法保存   
        'outtmpl': '%(id)s%(ext)s' 
    }
    
    with youtube_dl.YoutubeDL(ydl_opts) as ydl:
        ydl.download([youtube_url])
```

## 小结

把上述的方法都熟悉了，已经能解决大部分需求了。播放列表下载完，更进阶的处理包括添加章节信息等等，此处不展开。对于经常需要下载的同学，还可以设定命令别名，如

```bash
# .bashrc
alias youdl='youtube-dl --proxy "socks5://x.x.x.x:port" --write-sub --sub-format "ass/srt/best" --convert-subs "srt"'
```

以后使用时只需要敲 *youdl* 即可。

再有按上述方法下载好文件后，发现并未自己帮你合并视频、音频时，则手动处理下吧，把它们封装到 *mkv容器*。

如果觉得怎么又视频，又音频，又说合并为视频文件之类的很难懂，则请记住以下等式

```bash
# 容器 = 视频轨 + 音频轨 + 字幕轨
mkv = 视频1 + 视频2 + 音频1 + 音频2 + 字幕1 + 字幕2
```

mkv是一个容器，把3种类型的轨道全装在一起，才会类似「国粤双语双字幕」这类的mkv视频文件。想了解更多 mkv 封装的内容，可参考 [qBittorrent下载之道](/2018/05/10/qbittorrent-usage) 。

好吧，你可以下山去浪了少年，enjoy yourself !!

### 更新记录

> 2018-05-21
> 增加 `脚本调用` 小节
