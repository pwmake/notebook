---
layout: post
title: jekyll教程01 - 使用说明
tags: [Jekyll]
feature-img: "https://images.pexels.com/photos/614117/pexels-photo-614117.jpeg?cs=srgb&dl=abstract-achievement-bright-614117.jpg&fm=jpg"
---

本文以最简单的语言来描述 `jekyll` 以及如何使用它，每个部分均会给出对应的官方文档链接，并把需要特别注意的地方标注出来，目的是让你快速上手。你可以把它理解成是辅助学习的提纲，而不是详细的官方文档翻译，嘿嘿！

<!--more-->

## 概览

`Jekyll` 是一个简单的博客形态的静态站点生产机器，说人话是用来生成静态站点的工具，其主要涉及以下几个元素：

1. `markdown` ： 博客文章是 *markdown* 格式
2. `liquid` ：一种轻量级的模板语言
3. `ruby` ： 使用 *ruby* 语言来进行转换操作
4. `sass` ： CSS 预处理
5. `yaml` ： 配置文件采用此格式，易配置，易于人类阅读

举个例子，假设要做五仁月饼，你把所需要的材料都准备好，全部扔到 `生产机器` 上，它自动把你把五仁月饼做好，嘿嘿！那以上要素则代表五仁材料，`jekyll` 则是 `生产机器` ，自动帮你处理好，转换成可以直接放在 `web服务` 目录下的静态网站文件。

jekyll文档资料目前有两个渠道，一是 [官方英文文档](http://jekyllrb.com/docs/themes/) ， 二是 [中文翻译文档](http://jekyll.com.cn/docs/home/) ，建议以 `英文文档` 为主，如果英文确实读不懂，可以先看 **中文**  的，再看 **英文**， 以便及时了解新功能。

## 安装与使用

### 安装

官方的 [安装](http://jekyllrb.com/docs/installation/) 文档过于简单，等于是没说，此处展开说说，以 *ubuntu 16.04* 为例子

```bash
$ sudo apt-get update

# 安装ruby
$ sudo apt-get install ruby ruby-dev

# 安装nodeJS
$ sudo apt-get install nodejs

# 安装make gcc gcc+ ，安装jekyll需要使用
$ sudo apt-get install make gcc gcc+ build-essential

# 安装jekyll
# 会自动安装liquid，kramdown, yaml, sass和rouge
$ sudo gem install jekyll bundler
# 如果上述命令很久都没反应，请更换gem源
$ sudo gem sources --remove http://rubygems.org/
$sudo gem sources -a https://ruby.taobao.org/

# 检查下python 和 ruby的版本
# python >= 2.7 ; ruby >= 2.4
$ ruby -v
$ python -V
```

`gem` 命令相当于 *python* 的 `pip` ，*js* 的 `npm` ，是一种包「模块」管理器。

### 使用

成功安装后，建立一个新项目来熟悉如何使用

```shell
# 创建新项目，使用默认选项
$ jekyll new test-site
```

*jekyll* 常用的启动命令请查看 [Basic Usage](http://jekyllrb.com/docs/usage/) ，此处以变更端口为例子示范

```shell
# 默认用法 127.0.0.1 4000端口
$ jekyll serve

# 更改为 全局监听 9167端口
$ jekyll serve -H 0.0.0.0 -P 9167
```

更多详细的启动参数，请参阅 [configuration](http://jekyllrb.com/docs/configuration/) ；启动参数亦可写在 主配置文件 `_config.yml` 里头。

其次是 *jekyll* 项目的 [目录结构](http://jekyllrb.com/docs/structure/) ，要大概明确，每个目录是用来做什么的，目录均以 `_` 开头。

## 写文章

### 头信息

写 `markdown` 格式的文章，需要添加添加 [头信息](http://jekyllrb.com/docs/frontmatter/) 「置于md文件开头位置」，以告知 *jekyll* 在转换时，如何处理文章的某些属性。

以一个完整的 *头信息* 讲解

```yaml
---
layout: 采用何种布局, 必填
title: 此篇文章的中文标题, 必填
permalink: 定义链接样式，可在主配置文件中定义全局链接样式
published: 是否出版，即是否转换成html文件，默认true，false为不出版
date: 日期格式，不设定此项，则默认使用md文件名的日期，后面会讲到
category: 分类，单个，
categories: 分类，多个，列表的形式 ['a', 'b']
tags: 标签，单个or多个，后者也要以列表的形式撰写
keywords: 关键词，此项为自定义的
---
```

不必把上述所有的 头信息 都写上，按需选择，除了必填的两项，`categories` 和 `keywords` 建议填写，特别是需要把 *keywords* 添加到 `SEO` 的。

当某篇文章撰写了上述的 头信息，你可以理解成它最终成生成一个 `博客文章` 页面，那这个页面 `page` 的相关属性在模板语言中可表示为

```html
{% raw %}
<h1>{{ page.title }}</h1>
<p>关键词为 {{ page.keywords }}</p>
{% endraw %}
```

自定义的 *头信息* `keywords` 可直接访问。

最后是 `分类 category` 与 `标签 tag` 的区别，[如何使用它们](https://www.wpsitecare.com/wordpress-categories-vs-tags/) 呢？

1. 分类是一个大类，标签可以是其的子集
2. 标签更多是服务于 `SEO` 
3. 不同的站点有不同的用法

例如`A项` 有 早餐、午餐 和 晚餐，`B项`有 面点、米饭 等等。如果你想强调 三餐，则`A项` 设定为 `分类`， `B项`设定为`标签` ，反之，嘿嘿！

### 文章主体

首先是`文件名格式` 要求，必须是

```shell
year-month-day-title.markup
```

例如

```shell
2017-09-06-chapter-one-jekyll.[md|markdown]
```

后缀名可以是 `.md` 或者 `.markdown` ，日期格式必须写正确。

其次是 `静态文件` 的添加，可以使用

```shell
{% raw %}
# 站内图片文件
![我是图片]({{ site.url }}/assets/screenshot.jpg)

# 站内常规文件
[我是文件]({{ site.url }}/assets/iamfile.pdf)

# 站外文件
# 直接使用markdown的格式添加链接咯
{% endraw %}
```

接着是 `站内链接` 的添加，可以使用

```html
{% raw %}
{{ site.baseurl }}{% link _posts/2016-07-26-name-of-post.md %}
{{ site.baseurl }}{% link news/index.html %}
{{ site.baseurl }}{% link /assets/files/doc.pdf %}
{% endraw %}
```

最后是 `摘要分隔符` 

```html
{% raw %}
<main class="post-content">
  {{ post.excerpt }} <!-- 使用 excerpt 自动截取摘要 -->
</main>
{% endraw %}
```

截取的摘要为第一段 `<p></p>` 的内容，如果想自定义 分隔符，可在文章中这样操作

```markdown
---
excerpt_separator: <!--more-->
---

我是摘要部分
<!--more-->
excerpt_separator亦可在主配置文件 _config.yml 全局定义
```

## 创建新页面

单个页面的话，可直接放在项目根目录，多个页面的话，则可以自定义 `page` 目录，存放所有的固定页面，例如 `关于我` 页面。

页面可以是 `markdown` 文件，如

```markdown
---
title: 关于我
layout: page
permalink: /about/
---

这是关于我的固定页面
```

页面也可以是 `.html` 文件，如

```html
---
title: 关于我
layout: page
permalink: /about/
---

<p>
  这是关于我的固定页面，还可以使用 liquid 的模板语言来生成其他内容噢
</p>
```

## 静态资源

静态资源的存放在目录 `assets` 

```shell
$ tree assets -L 1
assets
├── css		# 主要样式
├── fonts	# 图标
├── img 	# 图片
└── js		# javascript

4 directories, 0 files
```

 `特别注意`： 敲黑板重点了，嘿嘿！`sass`  的处理，涉及知识点为 [theme 主题](http://jekyllrb.com/docs/themes/) 。

```shell
$ tree _sass
_sass
├── june
│   ├── _base.scss		# 定义通用基础样式
│   ├── _highlight.scss	# 定义代码高亮样式
│   └── _layout.scss	# 定义页面布局样式
└── june.scss 			# 主题名字，定义基础变量

1 directory, 4 files
```

以上是 `sass` 的目录结构，如果有按步骤来新建一个 *jekyll* 项目，默认会使用 `minima`主题 ，其路径为

```shell
# 因系统而异
$ sudo ls /usr/local/lib/ruby/gems/2.4.0/gems/minima-2.1.1
```

可切换过去，了解目录结构「虽然上述已经解释了」。

接着设置 预处理 后的 `.css` 在放在哪里了

```shell
# 假设最终的.css 为 assets/css/main.scss
$ cat > assets/css/main.scss << EOF
---
---
@import "june";
EOF
```

如此，开启 *jekyll* 服务后，则自动生成 `assets/css/main.css` ，更多详细参阅 [assets scss](http://jekyllrb.com/docs/assets/) 。

## 变量

指在 `liquid模板语言` 内的变量，在 *jekyll* 里有以下几个全局变量

| 变量        | 描述                            |
| --------- | ----------------------------- |
| site      | 网站变量，在主配置 `_config.yml` 设置的变量 |
| page      | 页面变量，即页面内的变量 和  文章头信息中自定义的变量  |
| layout    | 布局变量，主要是文章头信息中，指定用哪种布局        |
| content   | 主体内容，替代为文章内容 或者 页面内容          |
| paginator | 分页器变量，保存分页信息                  |

需要深入研究的三大变量是：`site` 、`page` 和 `paginator` 。如首页、归档页的模板该怎样写，考验你对这三大变量的熟悉程度。详细的请看 [variables 变量](http://jekyllrb.com/docs/variables/) 。

## 数据文件

除了上述的变量外，你还可以设置自定义数据在 `liquid` 模板中使用，即 [Data Files](http://jekyllrb.com/docs/datafiles/) 。

支持 `YAML` 、`JSON` 和 `CSV` 三种格式，个人推荐 `yaml` ，嘿嘿。放在指定目录 `_data` 下即可，例如

```yaml
# _data/global.yml 全局数据
title: My Site
```

在模板中调用时，只需

```html
{% raw %}
<h1>我的网站标题是：{{ site.data.global.title }}</h1>
{% endraw %}
```

`P.S`：还有一种与 *data files* 类似的 —— [Collections](http://jekyllrb.com/docs/collections/) ，用于保存某些静态信息的，例如开发人员信息等等。

## 部署

把 *jekyll* 项目部署到  *github pages* ，则可以拥有一个免费的 `username.github.io` 域名的静态网站，部署方法可参考中文的 [github pages部署](http://jekyll.com.cn/docs/github-pages/) 文档，如果你有建站的基础知识，可以自行部署在自己的服务器上，使用自己的域名。抑或谷歌一下，查询别人分享的部署方法。

官方的 *github pages* 部署文档务必先阅读。理清两种不同的目录结构的组织方式，判断是否需要修改 `_config.yml` 中的 *baseurl* 。如果使用自定义域名，又是另一种处理方式。

## 小结

纵观全文，*jekyll* 如何使用估计已经心中有数了。使用默认主题，增加自己想要的文章，再部署到 *github pages* 上就可以拥有自己的网站了。

那如果我想要自定义某些内容呢？嘿嘿！咱们留到第2节继续讲。

## 参考链接

> [jekyll中文教程](http://jekyll.com.cn/docs/home/)
>
> [jekyll官方英文教程](http://jekyllrb.com/docs/themes/)
