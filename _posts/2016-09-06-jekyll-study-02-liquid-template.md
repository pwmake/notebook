---
layout: post
title: jekyll教程02 - liquid模板
tags: [Jekyll]
feature-img: "https://images.pexels.com/photos/614117/pexels-photo-614117.jpeg?cs=srgb&dl=abstract-achievement-bright-614117.jpg&fm=jpg"
---


使用 *jekyll*  ，除了简单的 `markdown` 和 `yaml` 之外，重点是 `Liquid Template Language` —— liquid模板语言。它能够让你自定义所有页面的结构。如果之前使用过其他模板语言如 `jinja2` ，`django template language` 等等，基本上是秒入门，只需要区分各自异同之处即可使用。接下来，将对 *liquid* 和 *jekyll* 两者间的关系展开，重点讲前者。

## Liquid

> Liquid 是一门开源的模板语言，由 [Shopify](https://www.shopify.com/) 创造并用 Ruby 实现。它是 Shopify 主题的骨骼，并且被用于加载店铺系统的动态内容。

引用文末的中文翻译。鉴于本人目前只使用过 *django* 的模板语言，仅以此作对比。两者的不同之处是，**但凡django模板能实现的，liquid也能实现**， 后者还额外支持 `变量` 这一杀手级的特性。*django* 模板设计的初衷是为了让动态内容生成足够简单，同时也让前端人员易于阅读，仅支持持简单的逻辑处理，避免把它设计为一门语言，降低使用复杂度。*django* 允许用户撰写自定义的过滤器和标签，这是其`优势`之处。

所有模板语言大体可分为两大基本要素： `Tags 标签` 和 `Filters 过滤器` 。 

以下挑几个认为重点的来讲解。

### basic基础

有`数组对象` 这一概念，可通过 *迭代* 或者 *索引* 来访问，如

```html
{% raw %}
<!-- if site.users = "Tobi", "Laura", "Tetsuro", "Adam" -->
<!-- 迭代 -->
{% for user in site.users %}
  {{ user }}
{% endfor %}

<!-- 索引 -->
{{ site.users[0] }}
{% endraw %}
```

无法直接初始化一个数组，可通过 `spli` 过滤器分割一个字符串为子字符串数组，如

```html
{% raw %}
{% assign beatles = "John, Paul, George, Ringo" | split: ", " %}

{% for member in beatles %}
  {{ member }}
{% endfor %}
{% endraw %}
```

`控制输出的空白符`，部分环境下无法使用。

### tags标签

变量标记 ，用于创建新的变量。

`assign` 用于声明新变量，如

```html
{% raw %}
{% assign my_variable = false %}
{% if my_variable != true %}
  This statement is valid.
{% endif %}
{% endraw %}
```

`capture` 用于将开始与结束标记之间的字符串捕获后赋值给一个变量，该变量是`字符串` ，如

```html
{% raw %}
{% capture my_variable %}I am being captured.{% endcapture %}
{{ my_variable }}
{% endraw %}
```

两者组合可以创建复杂的变量。

默认是没有 `include` 标签  ； 输出原始字符，即不解析 *liquid* 标签 使用 `raw` 标签。

### filters 过滤器

过滤器数量太多，不逐个介绍了。日常使用多的主要是

1. append —— 增加
2. date —— 日期时间
3. default —— 设定默认值
4. escape —— 转义
5. reverse —— 反转，迭代时逆序
6. split —— 分割

## Jekyll

`liquid` 有3大分支，*jekyll* 占据了一个，其额外增加了自定义的过滤器和标签，使用详情查阅 [jekyll template](http://jekyllrb.com/docs/templates/) 。

**过滤器**： 添加了 `where` 条件过滤 和 `group` 分级过滤。

**标签**：     添加了  `include` 包含标签，还有 `link` 站内资源的链接标签。

除此之外，*jekyll* 还提供了 [永久链接](http://jekyllrb.com/docs/permalinks/) 、[分页器](http://jekyllrb.com/docs/pagination/)、[插件](http://jekyllrb.com/docs/plugins/)、[主题](http://jekyllrb.com/docs/themes/) 的自定义选项，前两者使用频率较高。

## include标签

- 包含 `_includes` 目录下的文件

```html
{% raw %}{% include footer.html %}{% endraw %}
```

- 包含 任意目录下的文件，不能使用 `../` 目录

```html
{% raw %}{% include_relative somedir/footer.html %}{% endraw %}
```

- 包含头信息的变量

```yaml
---
title: My page
my_variable: footer_company_a.html
---
```

使用时

```html
{% raw %}{% include {{ page.my_variable }} %}{% endraw %}
```

- 包含传递的变量

```html
{% raw %}
<div markdown="span" class="alert alert-info" role="alert">
<i class="fa fa-info-circle"></i> <b>Note:</b>
{{ include.content }}
</div>
{% endraw %}
```

使用时额外带待传递的变量名为参数

```html
{% raw %}{% include note.html content="This is my sample note." %} {% endraw %}
```

`content` 内容会替换掉模板内的 `include.content` ；可传递多个参数，嘿嘿！

content 还可以传递 `_data` 中的 `yaml` 文件，接着在模板中迭代数据，好强大的说。

## 小结

*jekyll* 在原有 *liquid* 的基础上扩展了专有的过滤器和标签，与此同时，原有的 *去除空白符* 标签却不兼容使用。

*jekyll* 能给予的自定义选项，要么是 *模板*，要么是 *插件* ； 前者对应 `liquid` ，后者对应 `ruby` 。如果需要深入研究，可自行扩展。

## 参考链接

>  [liquid 中文教程](https://liquid.bootcss.com/)
>
> [liquid 英文教程](https://shopify.github.io/liquid/)