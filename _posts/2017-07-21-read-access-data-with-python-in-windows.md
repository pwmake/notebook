---
layout: post
title: win使用python读取access数据
date: 2017-07-21 12:57:46.057000
tags: [HowTo]
feature-img: "assets/img/pexels/startrails.jpg"
---

工作中有如题需求，遂捣鼓了好一阵子，啧啧，先来一句话总结，用 `windows系统` 真不是一般地蛋疼啊，整个问题搞下来，最浪费时间的不是敲代码，而是理清系统环境的坑。

<!--more-->

## 32bit or 64bit？

这尼玛是最蛋疼的地方。先说明下需求：

- 从access导出考勤数据，再导入到远程的mysql数据库中

当中需要涉及的内容有：

1. Python
2. office [access]
3. pyodbc模块
4. mysql模块

### Office

瘟系统是32位的最好办，基本没啥烦恼；64位的话则需要理清下逻辑了。

首先 office 的版本，即使你在64位系统上安装，除非你手动选了安装64位的office，否则默认还是会安装32位版的，因为「巨硬」自身也推荐用户使用32位版的 *office* 。此外，非原版的瘟七系统，有些改版的默认会自带office，并且无法删除的，这种更蛋疼了，还好，局面仍然是安装32位的office。

另外需要安装 [access 2010数据库引擎可再发行程序包](https://www.microsoft.com/zh-cn/download/details.aspx?id=13255) ，因为是32位的office ，你也只能安装32位的引擎包了，呵呵。

### python

同上，使用 `pyodbc` 的话，仍然得跟着上面走，*python* 也只能安装32位版的

#### mysql

瘟系统上安装 「mysqlclient」模块只能手动编译，无法像 *linux* 系统那样直接 *pip* 。其中涉及到什么鬼 *VC++* 之类的艹蛋玩意。所幸，有大神提供[编译好的版本](http://www.lfd.uci.edu/~gohlke/pythonlibs/) ，找到对应的 *python* 和 操作系统位数即可。仍然是32位噢，亲！

## access 选择日期范围

环境搞好之后，可以动手敲代码了。读取 *access* 数据库还得有固定的格式，如下所示

```Python
import pyodbc

conn = pyodbc.connect(
        'DRIVER={Microsoft Access Driver(*.mdb)};DBQ=%s;UID=%s;PWD=%s' % (db, user, passwd)
# 如果是新格式的话，仅64位体系下的
# DRIVER={Microsoft Access Driver(*.mdb)}
```

接着是选择日期范围，*access* 最蛋疼之处居然是选择日期时，还他喵的有自己的特色。

```sql
select num, work_date from table where work_date between #2017-06-01# and #2017-06-12#;
```

看到没有，是要加 *#* 号滴，啧啧……啊……!!!
