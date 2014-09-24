---
layout: post
category : Django
title : "你的第一个Django应用(part 1)"
tagline: "Supporting tagline"
tags : [python, Django]
---

[https://docs.djangoproject.com/en/1.7/intro/tutorial01/](https://docs.djangoproject.com/en/1.7/intro/tutorial01/)

该教程从头到尾创建一个投票应用，包含两部分：

 - 公开部分，允许人们查看投票，进行投票
 - 管理员部分，允许你增加、修改、删除投票

该教程适用于Django 1.7和Python 3.2或更高。如果你使用Python 2.7，那么你需要按照注释稍微调整示例代码。

##新建项目
命令行执行

    django-admin.py startproject mysite
    
>注意：项目名称避免使用Python和Django内建模块名称。比如：django或test(和Python内置包名冲突)

上面的命令会生成如下目录结构
	
	mysite/
		manage.py
		mysite/
			__init__.py
			settings.py
			urls.py
			wsgi.py
			
- 最外层的`mysite/`目录只是你项目的一个容器，它的名字跟Django无关，你可以随意重命名。
- `manage.py`是一个命令行工具
- 内层的`mysite/`目录是你项目真正的Python包。当你需要引入它里面的内容时，例如：`mysite.urls`

###建立数据库
首先要安装正确的[database bindings](https://docs.djangoproject.com/en/1.7/topics/install/#database-installation)，然后修改`DATABASES`里的 `default`项。
###### 
- ENGINE 可选值`django.db.backends.sqlite3`, `django.db.backends.postgresql_psycopg2`, `django.db.backends.mysql`, or `django.db.backends.oracle`或[其它](https://docs.djangoproject.com/en/1.7/ref/databases/#third-party-notes)
- NAME 数据库的名字

运行下面的命令在数据库中创建表

	python manage.py migrate
	
###开发服务器
运行项目

	python manage.py runserver

>对于每一个请求，如果有必要的话，开发服务器会自动重新加载Python代码。所以你不用重新启动服务器来看效果。但是一些动作比如：添加新文件不会触发重新加载，在这种情况下，需要你重新启动。

##新建models
>Projects 与 apps 的关系：一个项目可以有多个apps,一个apps可以存在于多个项目中。

Your apps can live anywhere on your [Python path](http://docs.python.org/tutorial/modules.html#the-module-search-path).

通过下面的命令创建你的app。

	python manage.py startapp polls

写Django应用的第一步就是定义你的models。

>理念：DRY Principle。

在我们的投票应用中会用到两个models：`Question`和`Choice`。如下：
	
	from django.db import models

	class Question(models.Model):
    	question_text = models.CharField(max_length=200)
    	pub_date = models.DateTimeField('date published')

	class Choice(models.Model):
    	question = models.ForeignKey(Question)
    	choice_text = models.CharField(max_length=200)
    	votes = models.IntegerField(default=0)



##Activating models

上面的一小段代码给了Django很多信息，用这些信息，Django可以：

- 创建数据表
- 创建访问数据的API

但是首先我们需要告诉我们的项目我们新建了polls app

>理念：Django apps 是可插拔的。

修改`mysite/settings.py`，将`polls`添加到`INSTALLED_APPS`。

现在Django知道了polls 的存在，运行下面的命令：

	python manage.py makemigrations polls
	
通过运行`makemigrations`,你告诉Django，你对models做了一些修改，并且想把修改保存为`migration`

Migrations 是Django保存你模型变化的文件。它被设计为可人工修改的格式。

`migrate`命令用来运行migrations,并自动操纵数据库。首先，我们想看看migration会执行哪些SQL

	python manage.py sqlmigrate polls 0001
	
该命令接受migration name 并返回对应的SQL

需要注意以下几点：

- 返回的SQL基于你的数据库设置
- 表名自动加上了app(polls)前缀，并且变成小写(可修改该行为)
- 自动加上了主键(可修改该行为)
- 自动给外键加上了`_id`后缀(可修改该行为)
- 并不会真正修改数据库

可以通过下面命令来检查项目是否存在问题

	python manage.py check
	
接下来，运行`migrate`,在数据库中创建新表

	python manage.py migrate
	
`migrate`命令取得所有还未应用的migrations(Django用一个特殊的数据库表django_migrations记录)，并在数据库中执行。
实际上就是把你对模型的修改同步到数据库中。

Migrations 非常强大，让你可以随时修改model，而不用删掉数据库重新建表。它可以更新你的数据库，同时不丢失数据。请记住，修改模型的三个步骤：

- 修改模型
- 执行 `python manage.py makemigrations`给对应的修改创建migrations
- 执行 `python manage.py migrate` 把修改应用到数据库

把创建和应用migration的命令分开的好处是你可以把migration文件通过版本控制提交，并和你的应用一起分发。使协作开发更容易。

 


##playing with the api
执行下面命令，调用Python shell

	python manage.py shell
	
我们直接使用`python`命令的原因是 manage.py 设置了`DJANGO_SETTINGS_MODULE` 环境变量，which gives Django the Python import path to your mysite/settings.py file.

启动shell 之后，你就可以探索[database api](https://docs.djangoproject.com/en/1.7/topics/db/queries/) 了
 
	>>> from polls.models import Question, Choice   # Import the model classes we just wrote.

	# No questions are in the system yet.
	>>> Question.objects.all()
	[]

	# Create a new Question.
	# Support for time zones is enabled in the default settings file, so
	# Django expects a datetime with tzinfo for pub_date. Use timezone.now()
	# instead of datetime.datetime.now() and it will do the right thing.
	>>> from django.utils import timezone
	>>> q = Question(question_text="What's new?", pub_date=timezone.now())

	# Save the object into the database. You have to call save() explicitly.
	>>> q.save()

	# Now it has an ID. Note that this might say "1L" instead of "1", depending
	# on which database you're using. That's no biggie; it just means your
	# database backend prefers to return integers as Python long integer
	# objects.
	>>> q.id
	1

	# Access model field values via Python attributes.
	>>> q.question_text
	"What's new?"
	>>> q.pub_date
	datetime.datetime(2012, 2, 26, 13, 0, 0, 775217, tzinfo=<UTC>)

	# Change values by changing the attributes, then calling save().
	>>> q.question_text = "What's up?"
	>>> q.save()

	# objects.all() displays all the questions in the database.
	>>> Question.objects.all()
	[<Question: Question object>]`
 
注意上面的最后一行，这种输出没有多大用处。我们修改Question model,给它添加一个 `__str__`()方法

	from django.db import models

	class Question(models.Model):
    	# ...
    	def __str__(self):              # __unicode__ on Python 2
        	return self.question_text

	class Choice(models.Model):
    	# ...
    	def __str__(self):              # __unicode__ on Python 2
        	return self.choice_text