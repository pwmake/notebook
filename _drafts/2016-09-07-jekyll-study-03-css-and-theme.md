---
title: jekyll教程03 - 样式与主题
layout: post
tags: [Jekyll]
feature-img: "https://images.pexels.com/photos/614117/pexels-photo-614117.jpeg?cs=srgb&dl=abstract-achievement-bright-614117.jpg&fm=jpg"
---

最后一节，咱们来聊聊 *jekyll* 的主题设计，即  `CSS样式` 。 *CSS* 全称是 [层叠样式表](https://zh.wikipedia.org/zh/%E5%B1%82%E5%8F%A0%E6%A0%B7%E5%BC%8F%E8%A1%A8) ，可自行了解，不展开。以原始方式编写 *CSS* 令人蛋疼不已，由于出现了样式预处理工具 —— `Sass` ，`Less` 等等。

<!--more-->

## SASS

语法主要有： `变量` ， `嵌套` ， `混合` ， `导入` ， `颜色` ， `继承` 等等。

### 变量

变量使用 `$` 来表示，可声明为 *颜色* ， *数值* 等等，区分作用范围，`{}`块体内优先，与编程语言的 *全局* 、*局部* 类似，当块体内无声明，则自动向上`寻找父级` ，直到找到为止。

声明变量时，有个特殊的用法 `!default` 

```scss
// 其他地方有定义 $brand-color 时，则以该定义为默认。
$brand-color: red !default;
```

与之相反的是 `!important` 

```scss
// 无论是否还有相同的定义，均以当前定义的值为准。
$brand-color: red !important;
```

### 嵌套

解决了原生 CSS 编译时的大量重复劳作，如

```scss
#content {
  article {
    h1 { color: #333 }
    p { margin-bottom: 1.4em }
  }
  aside { background-color: #EEE }
}

 /* 编译后 */
#content article h1 { color: #333 }
#content article p { margin-bottom: 1.4em }
#content aside { background-color: #EEE }
```

需要注意 *父选择器的标识符* `&` 的使用，如

```scss
article a {
  color: blue;
  &:hover { color: red }  // 把&去掉则为 a :hover了
}

/* 编译后 */
article a { color: blue }
article a:hover { color: red }
```

群组间亦可嵌套，如

```scss
.container {
  h1, h2, h3 {margin-bottom: .8em}
}

/* 编译后 */
.container h1 { margin-bottom: .8em }
.container h2 { margin-bottom: .8em }
.container h3 { margin-bottom: .8em }
```

以及属性的嵌套

```scss
nav {
  border: 1px solid #ccc {
  left: 0px;
  right: 0px;
  }
}

/* 编译后 */
nav {
  border: 1px solid #ccc;
  border-left: 0px;
  border-right: 0px;
}
```

还可以使用 `+ 同层相邻` 、`~ 同层全体` 、`> 直接子元素` 的层级嵌套。

### 混合

混合用于大段样式需要重复使用的情况， 如

```scss
/* 声明 混合器 */
@mixin rounded-corners {  // 使用 @mixin 声明混合器
  -moz-border-radius: 5px;
  -webkit-border-radius: 5px;
  border-radius: 5px;
}

/* 调用 混合器 */
notice {
  background-color: green;
  border: 2px solid #00aa00;
  @include rounded-corners; // 使用 @include 调用混合器
}
```

还可以给混合器传参，如

```scss
/* 传参 */
@mixin link-colors($normal, $hover, $visited) {
  color: $normal;
  &:hover { color: $hover; }
  &:visited { color: $visited; }
}

/* 传默认参数 */
@mixin link-colors($normal, $hover: black, $visited: red) {
  color: $normal;
  &:hover { color: $hover; }
  &:visited { color: $visited; }
}
```

### 继承

可以继承一段样式，也可继承某个元素

```scss
//通过选择器继承继承样式
.error {
  border: 1px red;
  background-color: #fdd;
}
.seriousError {
  @extend .error;
  border-width: 3px;
}

// 继承某个元素
.disabled {
  color: gray;
  @extend a; // 继承 a 默认的样式
}
```

### 计算

简单的四则运算

```scss
$unit-length: 10px;

div {
  height: $unit-length * 2;
}
```

### 颜色

```scss
lighten(#cc3, 10%) // #d6d65c
darken(#cc3, 10%) // #a3a329
grayscale(#cc3) // #808080
complement(#cc3) // #33c
```

### 导入

```scss
@import "path/file.scss";
```

### 条件

```scss
// 单if
p {
  @if 1 + 1 == 2 { border: 1px solid; }
  @if 5 < 3 { border: 2px dotted; }
}

// 添加else
@if lightness($color) > 30% {
  background-color: #000;
} @else {
  background-color: #fff;
}
```

### 注释

标准注释，编译后保留在 *css* 文件

```scss
/* comment */
```

单行注释，编译后不保留

```scss
// I am comment
```

重要注释，即使压缩后仍在

```scss
/*!
 我很重要
*/
```

## jekyll 默认主题

*jekyll* 默认主题是 `minima` ，查看其目录结构

```shell
$ tree _sass assets
_sass
├── minima
│   ├── _base.scss  # 基础样式
│   ├── _layout.scss  # 布局样式
│   └── _syntax-highlighting.scss # 代码高亮
└── minima.scss # 全局变量声明
assets
└── main.scss  # 最终生成的 css文件所在

1 directory, 5 files
```

仔细阅读其组织形式，在 `minima.scss` 中把所有的样式集中起来，最后统一编译成 `main.css` 。

### calc

着重讲下这个 `calc` ，计算宽度值时，务必减去水平方向的边距值，避免撑破父级元素。

```scss
@include media-query($on-palm) {
  .footer-col {
    float: none;
    width: -webkit-calc(100% - (#{$spacing-unit} / 2));
    width:         calc(100% - (#{$spacing-unit} / 2));
  }
}
```

## LESS

`less` 与 `sass` 类似，同样拥有 `变量` ， `混合` ，`继承` 等元素，稍微有点区别，上手非常快。

`bootstrap` 前端工具架构，使用的是 `less` ，其庞大的样式文件组织结构非常值得学习。

*less* 与 *sass* 两者的区别是

| 选项   | SASS                          | LESS               |
| ---- | ----------------------------- | ------------------ |
| 变量   | 使用 $ 声明变量                     | 使用 @ 声明变量          |
| 混合   | @mixin xxx； 调用时为 @include xxx | .xxxx ； 调用时为 .xxxx |
| 继承   | 使用 @extend                    | 使用 .xxxx ； 与混合类似   |
| 特色   | 拥有条件、循环 和 自定义函数               | 可使用数学函数，JS表达式      |
| 依赖   | ruby语言                        | node.js            |

其他则比较类似，例如 `颜色` ，部分颜色计算公式有少许差别而已，不逐一罗列。

## 安装与使用

### sass

安装依赖环境

```shell
$ sudo apt install ruby ruby-dev
```

安装 *sass* 

```
$ sudo gem install sass
```

使用 *sass* 

```shell
# 打印编译后的css样式
$ sass test.scss

# 生成指定的 css 文件
$ sass test.scss test.css

# 生成压缩后的 css 文件
$ sass --style compressed test.scss test.css

# 监测文件
$ sass --watch input.scss:output.css
```

### less

安装依赖环境

```shell
$ sudo apt install nodejs
```

安装 less

```shell
$ sudo npm install less
```

使用 less

```shell
# 打印编译后的 css 样式
$ less test.less

# 生成指定的 css 样式
$ less input.less > output.css

# 生成压缩的 css 样式
$ less  --clean-css input.less > output.css
```

## 尾巴

较之 `less` ，个人更喜欢使用 `sass` ，功能与使用灵活性上更强。

这个快速的 `jekyll` 教程到此完结，仅给出了一个简单的实践解释，一来供他人使用，二来供日后快速回顾，算是对自己学习成果的一个交待吧！



## 参考链接

> [CSS W3schools 教程](http://w3schools.bootcss.com/css/default.html)
>
> [W3.CSS W3schools 教程](http://w3schools.bootcss.com/w3css/default.html)
>
> [Sass 基础教程](http://www.sasschina.com/guide/)
>
> [Less 基础教程](http://www.bootcss.com/p/lesscss/)
>
> [SASS用法指南](http://www.ruanyifeng.com/blog/2012/06/sass.html)
>
> [Less 快速入门](http://less.bootcss.com/)
>
> [CSS3 的 calc 使用](https://www.w3cplus.com/css3/how-to-use-css3-calc-function.html)