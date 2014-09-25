---
layout: post
category : Django
title : "你的第一个Django应用(part 1)"
tagline: "Supporting tagline"
tags : [python, Django]
---

[https://docs.djangoproject.com/en/1.7/intro/tutorial02/](https://docs.djangoproject.com/en/1.7/intro/tutorial02/)

##创建admin user

	python manage.py createsuperuser

##启动开发服务器

	python manage.py runserver

##进入后台
将settings.py里的`LANGUAGE_CODE` 改为 `'zh-cn'`，可以让后台翻译成中文

##让投票程序后台可编辑
我们需要告诉admin，`Question`对象有一个admin 接口。编辑polls/admin.py 文件

	from django.contrib import admin
	from polls.models import Question

	admin.site.register(Question)


##探索后台

##自定义后台表单



##添加相关对象

##自定义后台chage list

##自定义后台外观

##自定义后台首页