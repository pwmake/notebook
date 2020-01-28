---
title: django基于类的视图详解
layout: post
date: 2017-04-13 15:41:33
tags: [Reference]
feature-img: "https://images.pexels.com/photos/248515/pexels-photo-248515.png?cs=srgb&dl=code-coding-computer-248515.jpg&fm=jpg"
---

对于入门级别而言，在视图中编写简单的函数能够满足基本需求，如访问文章等等。可一旦代码的量上来了，随着功能的增加，在视图中难免会出现大量的重复代码，此时考验的不是思维能力，而是敲代码人员的耐心了。基于类的视图的出现，解救了这一切。本文以撰写一个简单的博客为例子，讲解基于类的视图如何在实战在应用。

<!--more-->

## 博客模型

首先来定义博客文章的模型

```python
# views.py
from django.core.urlresolvers import reverse
from django.db import models
from uuslug import uuslug
from django.utils.timezone import now as dj_now
import re

class Article(models.Model):
    title = models.CharField(max_length=50, verbose_name=u'中文标题')
    author = models.CharField(max_length=50, verbose_name=u'作者')
    slug = models.SlugField(editable=False)
    views = models.IntegerField(default=0, verbose_name=u'阅读数')
    body = models.TextField(verbose_name=u'文章内容')
    created = models.DateTimeField(default=dj_now, verbose_name=u'创建日期')
    updated = models.DateTimeField(auto_now=True, verbose_name=u'更新日期')

    def save(self, *args, **kwargs):
        tmp = self.title
        result = re.findall(re.compile('.*?(\w+).*?'), self.title)
        for item in result:  # 在英语与数字后面自己添加空格
            tmp = tmp.replace(item, item+' ')
        self.slug = uuslug(tmp, instance=self)
        super(Article, self).save(*args, **kwargs)

    def __unicode__(self):
        return u'%s - %s' % (self.author, self.title)

    # 定义获取 绝对url 的方法
    def get_absolute_url(self):
        path = reverse('article_list', kwargs={'slug': self.slug})
        return path
```

使用 [uuslug](https://github.com/un33k/django-uuslug) 模块，让实例在保存时，自动将中文标题转换为拼音，详细使用可参考项目主页。加入正则过滤，为的是把标题中的数字、英文自动添加多一个空格，让显示效果更为人性化。

```python
from uuslug import slugify
txt = '我又老1岁了，sad啊'
slugify(txt)
u'wo-you-lao-1sui-liao-sada'
# 理想中的输出应该是
u'wo-you-lao-1-sui-liao-sad-a'
# 因此添加上述的正则过滤
```

创建日期使用 *default=dj_now* ，则文章在初次生成时会自行建立。

更新日期使用 *add_now=True* ，则文章实例每次执行save()方法时会自动更新。

## 基于类的通用视图

基于类的通用视图主要包含3种类型，分别是

- 显示视图
  1. ListView  显示有多少文章实例
  2. DetailView  显示每篇文章的详细
- 编辑视图
  1. FormView	表单视图，用于自定义的表单行为处理
  2. CreateView   创建实例视图 
  3. UpdateView  更新实例视图
  4. DeleteView   删除实例视图
- 日期视图
  1. ArchiveIndexView  所有文章归档
  2. YearArchiveView  按年份的文章归档
  3. 还有月、日、周、今日视图

## 显示视图

### ListView

假设博客首页显示文章的标题，以 *ListView* 形式来写的代码则为

```python
# views.py
from django.views.generic import ListView
from yourapp.models import Article
## 首页视图
class ArticleList(ListView):
    model = Article
    
# urls.py
url(r'^$', views.ArticleList.as_view()),

# 模板文件 YourApp/templates/YourApp/article_list.html
{% raw %}
{% extends "base.html" %}

{% block content %}
    <h2>Index</h2>
    <ul>
        {% for article in object_list %}
            <li>{{ article.title }}</li>
        {% endfor %}
    </ul>
{% endblock %}
{% endraw %}
```

这代码量非常感人啊。比之前用函数定义效率高太多了，嘿嘿！先不急，来看看它都做了什么

从哪个表中取数据？指定模型呗

```python
# views.py
class ArticleList(ListView):
    model = Article  ## 句式1
    query=Article.objects.all()  ## 与句式1等价
    
    ## 如果想要自定义更复杂的,可重写获取结果集的方法
    def get_queryset(self):
        if self.request.user.is_authenticated():
            query = Article.objects.all()
        else:
            query = Article.objects.filter('自定义的某些复杂过滤规则')
        return query
```

如何将结果传到模板呢？

```python
# views.py
class ArticleList(ListView):
    ...
    context_object_name = 'posts_list'  ## 定义queryset结果集在模板中的名字
    template_name = 'blog_index.html'
```

视图中查找出来的结果集默认名字是 `object_list`， 同时亦是模型名打头的 `article_list` ，可在视图中自定义。

而模板文件，默认是当前应用模板文件夹下的，如 *YourApp/templates/YourApp/article_list.html* ；同样以模型名开头，是list的则以 `_list`后缀结尾；可以自行在视图在定义。

### DetailView

首页搞后，接下来是每篇文章的详细内容，通过 *slug* 找到指定的文章，并显示其详细内容。 

```python
# views.py 
class ArticleDetail(DetailView):
    model = Article
    
# urls.py
url(r'^article/(?P<slug>\S+)$', views.ArticleDetail.as_view()),

# 模板文件（同上） YourApp/article_detail.html
{% raw %}
{% extends "base.html" %}

{% block content %}
    <h2>{{ object.title }}</h2>
	<p>{{ object.body }}</p>
{% endblock %}
{% endraw %}
```

同样是感人的代码量啊，啧啧！ *DetailView* 这种是查找单个对象的，默认是以 `PK值` 或者是 `slug`  来查找。因此上述代码才得以如此精简。该如何理解呢？从 *slug* 入手来理解

假设以文章标题来搜索文章，并且标题是唯一的，不存在重复，则此时的视图应该如何写呢？

```Python
# views.py 
class ArticleDetail(DetailView):
    model = Article
    slug_field = 'title'
    slug_url_kwarg = 'title'
    
# urls.py
url(r'^article/(?P<title>\S+)$', views.ArticleDetail.as_view()),
```

注意 *urlconf* 中的 *kwags* 关键字变成了 *title* ，但 *DetailView* 默认的 `def get_obejct()` 仍然使用 `slug` 为关键字进行搜索，敲重点来了。

```python
slug_field = 'title'  # 替换get_object()中的slug名，即变成get(title=xx)
slug_url_kwarg = 'title'  # 替换 urlconf中的关键字变成 slug, 即变成get(title=title)
```

 此处实为一个替换，正确说应该是重写了两个属性，以此达到重用，嘿嘿！

再顺便讲下如何获取 `urlconf`中 *kwarg* 的值。假设每次访问时，仅更新访问 *views* 而不更新 *updated* 。

```python
# views.py
class ArticleDetail(DetailView):
    model = Article

	# 重写获取结果集方法
    def get_queryset(self, **kwargs):
        query = Article.objects.filter(slug=self.kwargs.get('slug'))
        views = query[0].views + 1
        query.update(views=views)
        return query
```

倘若 *urlconf* 中并不是 *slug* 为关键字怎么办？形如

```python
# urls.py
url(r'^article/(?P<address>\S+)$', views.ArticleDetail.as_view()),
```

那好办，在视图中增加

```python
slug_url_kwarg = 'address'
```

假设模型中有记录访问时间的字段如 last_view ，则可以重写获取对象方法，令其当每次访问时，更新该字段

```python
from datetime import datetime
# views.py 
class ArticleDetail(DetailView):
    model = Article
    
    def get_object(self):
        object = super(ArticleDetail, self).get_object()
        object.last_view = datetime.now()
        object.save()
        return object
```

默认只传输查询到的对象到模板，如果还要传输另外的对象或者字符呢？假设传送访问者名字

```python
# views.py 
class ArticleDetail(DetailView):
    model = Article
    
    def get_context_data(self, **kwargs):
        context = super(ArticleDetail, self).get_context_data(**kwargs)
        context['request_user'] = self.request.user.username
        return context
```

## 编辑视图

### CreateView

先讲创建文章，撰写一篇新的文章，只需要填写标题和文章内容即可，其他均是自动创建，文章作者则以已登录的用户即可

```python
# views.py
from django.views.generic.edit import CreateView # 导入路径与上面有区别

class CreateArticleView(CreateView):
    template_name = 'YourApp/article_form.html' # 默认的模板文件名
    model = Article
    fields = ['title', 'body']

    def form_valid(self, form):  # 文章作者为登录者名字
        # form.instance代表创建的实例
        form.instance.author = self.request.user.username  
        return super(CreateArticleView, self).form_valid(form)

# urls.py
url(r'^create/$', views.CreateArticleView.as_view()),

# 模板文件 YourApp/article_form.html
{% raw %}
<form action="" method="post">{% csrf_token %}
    {{ form.as_p }}
    <input type="submit" value="Save" />
</form>
{% endraw %}
```

这简单粗暴得无法用语言来形容啊，连 `forms.py`也不用单独定义。

### UpdateView

文章发表之后，难免要小修小改，补充新的内容，内容与 *CreateView* 无异，只是名字上的区别

```python
# views.py
from django.views.generic.edit import UpdateView

class UpdateArticleView(UpdateView):
    template_name = 'YourApp/article_update_form.html' # 默认的模板文件名
    model = Article
    fields = ['title', 'tags', 'private', 'body']

    def get_success_url(self):  # 设置操作成功后自动跳转到哪个链接
        slug = self.request.resolver_match.kwargs['slug']
        return reverse('blog_article', kwargs={'slug': slug})
    
# urls.py
url(r'^edit/(?P<slug>\S+)/$', views.UpdateArticleView.as_view()),

# 模板文件， 与 CreateView中的一样
```

 如果没有重定向 创建/更新 成功后跳转到其他指定链接的需求，则不需要重写该方法，默认是跳转到模型中定义的 `get_absolute_url()`。 

### DeleteView

删除指定文章，同时接收 `POST` 和 `GET` 请求，前者则直接删除，后者则会跳转到一个确认删除的模板页面。

```python
# views.py
from django.views.generic.edit import DeleteView

class ArticleDelete(DeleteView):
    model = Article
    success_url = reverse_lazy('/')
    template_name = 'YourApp/article_confirm_delete.html' # 默认的模板文件名

# urls.py
url(r'^delete/(?P<slug>\S+)/$', views.ArticleDelete.as_view()),

# article_confirm_delete.html
{% raw %}
<form action="" method="post">
	{% csrf_token %}
    <p>Are you sure you want to delete "{{ object }}"?</p>
    <input type="submit" value="Confirm" />
</form>
{% endraw %}
```

### FormView

重头戏来了，嘿嘿！在学习表单时，咱们知道表单的创建方式有两种，

1. form.Form 属于自己动手，丰衣足实的类型，完全自定义表单的字段，clean() 和 save() 方法
2. ModelForm 属于懒人模式，自动创建， `上述的 CreateView 使用的即为此模式` 

下面以 创建文章 为例子讲解 `FormView` ，

手动撰写form

```python
# forms.py
from django import forms
from models import Article

class ArticleForm(forms.Form):
    title = forms.CharField(
        max_length=50,
        label=u'文章标题',
        widget=forms.TextInput(attrs={'class': 'form-control input-lg', 'placeholder': u'标题'}),
    )
    
    body = forms.CharField(
        min_length=10,
        label=u'内容',
        required=False,
        widget=forms.Textarea(),
    )
    
    def save(self, username, article=None):
        cd = self.cleaned_data
        title = cd['title']
        body = cd['body']
        author = username

        article = Article()
        article.title = title
        article.body = body
        article.author = author
        article.save()
```

在视图中使用 `FormView` 

```python
# views.py
from django.views.generic import FormView
from forms import ArticleForm

class PublishView(FormView):
    template_name = 'article_publish.html'
    form_class = ArticleForm

    def form_valid(self, form):
        form.save(self.request.user.username)
        return super(PublishView, self).form_valid(form)
```

是不是很简单呢亲？如果又想偷懒，又想表单中引入第三方模块来自定义化，怎么破？以 `ModelForm` 形式创建表单，添加额外的自定义信息，在 *CreateView* 中再添加个  `form_class = YourCustomizeForm` 即可。

还有 `AJAX` 形式的用法，详情查看末尾的官方链接。

## 日期视图

这节太简单了，看下官方文档即可，处理与日期相关的视图，方法有那么多，关键是思考如何运用吧！

## 权限处理

基于类的通用视图，权限处理则不能像函数时那样，增加添加个 `@login_required` 或 `@permisson_required` 就完事了。django 提供了一个 `method_decorator` 装饰器将函数式装饰器转换为方法类装饰器，额……这尼玛说起来好蛋疼，说人话是为了让类的每个实例都能使用到这个方法类装饰器。借助官方文档的例子来讲解

### 在dispatch方法为使用

```python
from django.contrib.auth.decorators import login_required
from django.utils.decorators import method_decorator
from django.views.generic import TemplateView

class ProtectedView(TemplateView):
    template_name = 'secret.html'

    @method_decorator(login_required)
    def dispatch(self, *args, **kwargs):
        return super(ProtectedView, self).dispatch(*args, **kwargs)
```

### 懒人做法

```python
@method_decorator(login_required, name='dispatch')
class ProtectedView(TemplateView):
    template_name = 'secret.html'
```

### 处理多个

```python
decorators = [never_cache, login_required]

# 默认按 decorators 列表中的顺序来执行
@method_decorator(decorators, name='dispatch') 
class ProtectedView(TemplateView):
    template_name = 'secret.html'

@method_decorator(never_cache, name='dispatch')
@method_decorator(login_required, name='dispatch')
class ProtectedView(TemplateView):
    template_name = 'secret.html'
```

## 关于其他零散的东东

若要查询每种视图包含什么属性、什么方法，以及每个方法怎么使用，可查看末尾的 [API](https://docs.djangoproject.com/en/1.10/ref/class-based-views/flattened-index/#createview) 那个链接。

基于类的通用视图还有两个比较给力的，直接在 `urls.py` 使用即可

```python
from django.views.generic import TemplateView  # 渲染指定静态Html文件
from django.views.generic import RedirectView  # 跳转到指定网站

urlpatterns = [
    url(r'^test/$', IndexView.as_view(template_name='static_html_file.html')),
    url(r'^drive_car/$', RedirectView.as_view(url='http://1024.com')),
]
```

例如把导航栏中友链均设置在 `urls.py` 中进行 *RedirectView* 跳转，避免每次修改模板文件。

## 小结

很奇怪的感觉……原本打算看django 1.8中文文档来学习，岂料难入法眼，翻看官方英文文档，瞬间透彻了许多。

## 参考链接

> [基于类的通用视图简介](https://docs.djangoproject.com/en/1.11/topics/class-based-views/intro/#id1)
>
> [内建的基于类的通用视图](https://docs.djangoproject.com/en/1.11/topics/class-based-views/generic-display/)
>
> [通用编辑视图](https://docs.djangoproject.com/en/1.10/ref/class-based-views/generic-editing/#django.views.generic.edit.FormView)
>
> [通用日期视图](https://docs.djangoproject.com/en/1.10/ref/class-based-views/generic-date-based/)
>
> [基于类的通用视图API详解](https://docs.djangoproject.com/en/1.10/ref/class-based-views/flattened-index/#createview)
>
> [基于类的视图处理表单](https://docs.djangoproject.com/en/1.11/topics/class-based-views/generic-editing/)
>
> [通用视图网友的总结](http://www.jianshu.com/p/67d034093c59)
>
> [网站URL标题自动转换slug](https://github.com/un33k/django-uuslug)