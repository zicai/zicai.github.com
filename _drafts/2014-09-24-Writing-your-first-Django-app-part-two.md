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

	from django.contrib import admin
	from polls.models import Question

	class QuestionAdmin(admin.ModelAdmin):
    	fields = ['pub_date', 'question_text']

	admin.site.register(Question, QuestionAdmin)

当你需要重写某个对象的后台选项时：创建一个model admin 对象，并将它作为`admin.site.register()`的第二个参数

当有很多的域时，你可能想将它们分成fieldsets

	class QuestionAdmin(admin.ModelAdmin):
    	fieldsets = [
        	('haha', {'fields': ['question_text']}),
        	('Date information', {'fields': ['pub_date']}),
    	]
	admin.site.register(Question, QuestionAdmin)

你可以给每个fieldset带上任意的HTML class 。Django 提供了一个`collapse`class ,它可以将fieldset 初始是折叠的。

	class QuestionAdmin(admin.ModelAdmin):
    	fieldsets = [
        	('haha', {'fields': ['question_text']}),
        	('Date information', {'fields': ['pub_date'],'classes':['collapse']}),
    	]
	admin.site.register(Question, QuestionAdmin)
	
##添加相关对象

现在我们有了Question 后台页面，我们想添加Choice后台页面，有两种办法：第一种就是直接注册：

	admin.site.register(Choice)
	
但是这样添加choice效率很低，我们想直接在Question页面，添加Choice，如下：

	class ChoiceInline(admin.StackedInline):
    	model = Choice
    	extra = 3

	class QuestionAdmin(admin.ModelAdmin):
    	fieldsets = [
        	('haha', {'fields': ['question_text']}),
        	('Date information', {'fields': ['pub_date']}),
    	]
    	inlines = [ChoiceInline]

	admin.site.register(Question, QuestionAdmin)

有个小问题就是，Choice的域占了很多屏幕空间，基于此，Django提供了一个tabular方式显示相关对象，修改如下：

	class ChoiceInline(admin.TabularInline):
		#...
		
改完之后，相关对象会展示的更紧凑，以表格形式展示
	
##自定义后台修改列表

默认情况下，Django显示每个对象的str()。但是有时候，我们想显示更多的信息。这要用到`list_display`admin选项。如下：

	class QuestionAdmin(admin.ModelAdmin):
		list_display = ('question_text', 'pub_date')
		
添加过滤器

	list_filter = ['pub_date']
	
添加搜索

	search_fields = ['question_text']
	

##自定义后台外观

新建一个template文件夹，修改`mysite/settings.py`,添加`TEMPLATE_DIRS`

	TEMPLATE_DIRS = [os.path.join(BASE_DIR, 'template')]
	
在template下新建admin文件夹，把Django源代码中默认的admin template目录(django/contrib/admin/template)下的模板文件(admin/base_site.html)拷贝过来

##自定义后台首页

操作如上，将Django源码中模板文件admin/base_site.html拷贝到文件目录下，进行修改