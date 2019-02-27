---
layout: post
title: 浅谈no www与网域规范
date: 2017-05-27 12:14:34.203000
tags: [Skill]
feature-img: "assets/img/pexels/startrails.jpg"
---


先表立场，我是忠实`no www` 使用者，即网站域名不添加 `www` 。为啥？从装逼的角度谈，简短的网址逼格尽显，在记忆方面效率会提升不少。如今`no www` 与 `www`共存的局面，有以下几种原因：

1. 从终端用户的体验来讲，*google.com* 比 *www.google.com* 少输3个字符，用户都是懒的。
2. 从历史的角度来讲，`www`的存在感已经根深蒂固在老网民心中了，造成其给人更正式的感觉，例如许多企业网站「仅针对中小企业」通常是使用 `www` ，而个人网站则是能简则简。
3. 从好奇心与经验主义的角度来讲，你无法阻止终端用户在两者之间相互跳，如 *example.com* 可以访问的话，那 *www.example.com* 呢？

那有没一种合适的方案解决上述的问题呢？答案是有的。

<!--more-->

## 重定向

首先确定你想使用的`首选域名` ，即是用 `www` 还是 `no www` 。

在网站服务器上配置 *apache* 的 `301 重定向`， 可写在 `.htaccess` 或者 具体的 `your_blog.conf` 网站配置文件中

- **no www 301重定向到 www**

```shell
# 301 跳转所有 no-www 到 www
RewriteEngine On
RewriteCond %{HTTP_HOST} ^[^.]+\.[^.]+$ [NC]
RewriteRule ^(.*)$ http://www.%{http_host}/$1 [R=301,NC]
```

如此一来，`example.com` 则会跳转到 `www.example.com` 。

- **www 301重定向到 no www**

```shell
# 301 跳转所有 www 到 no-www
RewriteEngine On
RewriteCond %{HTTP_HOST} ^www\.(.+)$ [NC]
RewriteRule ^(.*)$ http://%1/$1 [R=301,L] 
```

此时则为 `www.example.com` 自动跳转到 `example.com` 。

## 域名重定向

除了上面的301重定向之外，保险的做法是在域名解析上也做相应的跳转，以本站为例子，做跳转设置讲解。

- **CNAME** 

| Type         | Host | Value       | TTL       |
| ------------ | ---- | ----------- | --------- |
| CNAME record | www  | zshmobi.com | automatic |

- **URL Redirect Record**

某些域名提供商会额外提供 URL转发的记录，它比 `CNAME` 来得更直观易懂

| Type                | Host | Value               | TTL       |
| ------------------- | ---- | ------------------- | --------- |
| URL Redirect record | www  | https://zshmobi.com | automatic |

注意此时 `VALUE` 要写完整的域名地址噢。

域名重定向对新旧网站交替时做重定向非常有帮助。

## HTTPS

开启 `HTTP` 跳转到 `HTTPS` ，详见 [使用Let's Encrypt 开启HTTPS](/2017/07/21/enable-https-with-lets-encrpt) 的设定。

## 测试

命令行里使用 `curl` 进行测试

```shell
$ curl -i www.zshmobi.com
HTTP/1.1 302 Found
Server: nginx
Date: Wed, 02 Aug 2017 03:43:48 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 42
Connection: keep-alive
Location: https://zshmobi.com
X-Served-By: Namecheap URL Forward

<a href='https://zshmobi.com'>Found</a>.
```

或者直接在浏览器输入 `www.zshmobi.com` 看是否能跳转到 `zshmobi.com` 。

## 网域规范

当确定好 `no www` 和 `www` 之后，为搜索引擎解决了一大难题——`网域规范` ，在搜索引擎看来，`zshmobi.com/search/` 与 `www.zshmobi.com/search/` 是两个不同的URL。

> 为同一文档提供统一的URL
>
> 在访问同一内容时 , 为了防止一些用户链接到一个版本的URL而另一些用户链接到另一个版本(URL的不同可能会分散弱化该内容的声誉值) , 我们建议您在您页面的内部链接和结构中集中使用一个URL。如果您发现有用户通过不同的URL来访问同样的内容 , 您可以对不希望使用的URL设立301重定向到您所期望使用的URL。当重定向不起作用时 , 你也可以使用首选域URL或者rel=“canonical”的链接元素。
>
> ——摘自 《谷歌搜索引擎优化初学者指南》一书

现如今强制所有的 `www` 跳转到 `no www` 后，在谷歌站长工具中，设定首选域名为 `zshmobi.com` ，则搜索引擎只记录 `zshmobi.com` 的所有链接，提升网站的声誉值。

除此之外，建议在每个页面添加 `canonical` 链接元素，即

```Html
<link id="canonical" rel="canonical" href="xxxxx.com">
```

同样在新旧网站交替时，亦可使用

```Html
<!-- 在旧网站的页面上添加 -->
<link id="canonical" rel="canonical" href="新网站的地址">
```

此举让搜索引擎把旧网站的索引值全部指向新网站，等一段时间后，即可撤销旧网站了。

## 小结

感叹，装逼也需要文化啊！

## 参考链接

> [谷歌搜索引擎优化初学者指南](http://static.googleusercontent.com/media/www.google.com/en/us/intl/zh-cn/webmasters/docs/search-engine-optimization-starter-guide-zh-cn.pdf)
>
> [Redirecting and Remapping with mod_rewrite](http://httpd.apache.org/docs/current/zh-cn/rewrite/remapping.html)