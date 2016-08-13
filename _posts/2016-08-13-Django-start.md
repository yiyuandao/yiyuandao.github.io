---
layout: post
title: Django From scratch
---

## 0. django project

生成目录结构:
```
django-admin startproject mysite
cd mysite
django-admin startapp news
```
```
--|mysite/
----|manage.py
----|mysite/
------|__init__.py
------|setting.py
------|urls.py
------|wsgi.py
----|news/
------|models.py
------|views.py
------|urls.py
------|admin.py

```
## 1. Design your model
### 1.1 设计数据模型，自动生成数据库表
  mysite/news/models.py：

```
from django.db import models

class Reporter(models.Model):
    full_name = models.CharField(max_length=70)

    def __str__(self):              # __unicode__ on Python 2
        return self.full_name

class Article(models.Model):
    pub_date = models.DateField()
    headline = models.CharField(max_length=200)
    content = models.TextField()
    reporter = models.ForeignKey(Reporter, on_delete=models.CASCADE)

    def __str__(self):              # __unicode__ on Python 2
        return self.headline
```

### 1.2 更新数据库：

```
python manage.py migrate
```
### 1.3 Enjoy the free API
  通过django api 去操作数据库:CRUD
  ```
  python3
  from news.models import Reporter, Article
  # 查找所有reporter
  Reporter.objects.all()
  # 新建一个reporter
  r = Reporter(full_name='John Smith')
  # 保存到数据库
  r.save()
  # 查看reporter的ID
  r.id
  # 再次查看所有的reporter
  Reporter.objects.all()
  # 查看reporter的name属性
  r.full_name
  # django 提供的查找API
  Reporter.objects.get(id=1)
  Reporter.objects.get(full_name__startswith='John')
  Reporter.objects.get(full_name__contains='mith')
  Reporter.objects.get(id=2)

  # 创建一篇文章
  from datetime import date
  a = Article(pub_date=date.today(), headline='Django is cool',
      content='Yeah.', reporter=r)
  a.save()
  # 查找所有的文章
  Article.objects.all()
  # 通过文章去获取reporter信息
  r = a.reporter
  r.full_name
  # 通过reporte去获取文章的信息
  r.article_set.all()
  # 实现数据库的联合查询功能：join， 查找所有以john开头的report写的文章
  Article.objects.filter(reporter__full_name__startswith='John')
  # 修改reporter name
  r.full_name = 'Billy Goat
  r.save()
  # 删除reporter
  r.delete()
```
## 2. admin interface
 django 管理接口， 对app进行管理：CRUD
  mysite/news/admin.py：
```
from django.contrib import admin

from . import models

admin.site.register(models.Article)
```
生成admin用户：
```
django-admin createsuperuser
```
## 3. Design your URLs
mysite/news/urls.py
```
from django.conf.urls import url

from . import views

urlpatterns = [
    url(r'^articles/([0-9]{4})/$', views.year_archive),
    url(r'^articles/([0-9]{4})/([0-9]{2})/$', views.month_archive),
    url(r'^articles/([0-9]{4})/([0-9]{2})/([0-9]+)/$', views.article_detail),
]
```
## 4. Write your views
mysite/news/views.py
```
from django.shortcuts import render

from .models import Article

def year_archive(request, year):
    a_list = Article.objects.filter(pub_date__year=year)
    context = {'year': year, 'article_list': a_list}
    return render(request, 'news/year_archive.html', context)
```
## 5. Design your templates
mysite/news/templates/news/year_archive.html
```
{% extends "base.html" %}

{% block title %}Articles for {{ year }}{% endblock %}

{% block content %}
<h1>Articles for {{ year }}</h1>

{% for article in article_list %}
    <p>{{ article.headline }}</p>
    <p>By {{ article.reporter.full_name }}</p>
    <p>Published {{ article.pub_date|date:"F j, Y" }}</p>
{% endfor %}
{% endblock %}
```

mysite/templates/base.html
```
{% load staticfiles %}
<html>
<head>
    <title>{% block title %}{% endblock %}</title>
</head>
<body>
    <img src="{% static "images/sitelogo.png" %}" alt="Logo" />
    {% block content %}{% endblock %}
</body>
</html>
```

## 6. 测试
### 6.1 运行程序
```
python manage.py runserver 0.0.0.0:9000
```
### 6.2 展示页面
http://192.168.10.10:9000/
### 6.3 展示ADMIN页面
http://192.168.10.10:9000/admin


## 7. reference
1. [django-doc](https://docs.djangoproject.com/en/1.9/intro/overview/)
