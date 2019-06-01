---
title: 解决SVN提示UTF8乱码
layout: post
date: 2016-11-04 00:00:00
tags: [Skill]
feature-img: "assets/img/pexels/startrails.jpg"
---

最近工作上遇到了个算是奇葩的问题吧，使用SVN时，提示如下

```bat
svn: Can't convert string from 'UTF-8' to native encoding
```

网络上一番查找答案，大多是相互*借鉴* ，非常不满意；翻墙后查找，也效果甚微。当成功解决的喜悦迸发出来时，那股酸爽，简直不敢想象？得了，这逼装得有点过，下面请看老夫如何破解此案吧。

<!--more-->


## 背景交待

迫于外部因素，在`windows` 上搞自动化脚本，此前一直用着好好的SVN命令行，用来查询目录状态 ，突然间就提示 *UTF-8* 无法转换了。

首先，SVN自带的命令行应该不会有这个问题吧，我测试了下，正常。可怎么……不对，我系统上安装了*cygwin* ，是不是这货的问题。查看系统*PATH* 的默认搜索顺序，果然，调用的是*cgywin*里头的SVN命令，难怪呢……

把第一顺序修改为SVN自带的命令行目录，再次使用时，正常了，嘿嘿……

在*python* 调用时，也正常……

蛋是……当在*cgi* 中运行*python* 脚本时，脚本无法正常运行下去，查看日志得知，又他娘的提示UTF-8的错误，导致脚本运行中断了。为毛默认的SVN又跑回*cgywin*里头的呢？此处无解。

## 蛋疼的cygwin

网上查找的教程大多是指*linux*下的设置，甚少提及*windows* 。win默认的编码是`gb2312`， 而*cygwin* 下的则是`utf-8` 。

```shell
# zh_CN，设置无效
export LC_ALL=zh_CN.UTF-8 

# en_US，设置无效
export LC_ALL=en_US.UTF-8
```

忘记是哪篇国外的教程说，要做如下设置才可以 

```shell
export LC_ALL=C.UTF-8
```

 结果再测试 ，厉害了word天……成功解决了……/(T__T)\    