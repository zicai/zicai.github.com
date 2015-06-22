---
layout: post
category : lessons
title: "Meteor项目实战 -- Next 0.0.1"
tagline: "Supporting tagline"
tags : [meteor]
---

学了一段时间Meteor之后，着手做一个APP，关于时间管理的，取名 **Next**。

> Get things done , and do Next

同时，把开发过程尽可能详细的记录下来，分享给有需要的同学。前提是你有一些Meteor基础，至少要了解各种基础概念：集合、发布订阅、method、package等。

如果之前没了解过Meteor，可以快速过一遍[meteor教程](https://github.com/zicai/zicai.github.com/blob/master/_posts/2015-04-05-meteor-toturial.md)

Next 0.0.1版的目标是任务列表，四象限展示，可以新增、修改和删除任务。最后的效果如下图：(后面会慢慢提升她的颜值)

![Next 0.0.1 效果图](http://7xjw06.com1.z0.glb.clouddn.com/next 0.0.1 preview.png)

## 项目初始化

新建项目

	meteor create next
	
进入项目目录，并启动

```
cd next
meteor
```
打开浏览器，访问 http://localhost:3000，会看到自动生成的demo。

删除自动生成的文件，并建立项目目录结构

```
rm next.css next.html next.js
mkdir client public server lib
```

添加路由包`iron:router`；由于后面会用coffeescript写，所以还要加上`coffeescript`包

```
meteor add iron:router
meteor add coffeescript
```

## 第一条路由和模板
基本的骨架都有了，可以开始敲代码了。在client文件夹中新建router.coffee，内容如下：

```
Router.route('/', ->
  @render("Home")
)
```

浏览器中显示了一条错误信息：找不到home模板

	Couldn't find a template named "Home" or "home". Are you sure you defined it?

接下来定义home模板，在client文件夹中新建templates文件夹,在里面新建home.html,内容如下：

```
<template name="home">
    welcome to home
</template>
```

此时浏览器显示：

	welcome to home


## 新建collection
在lib文件夹中新建collections文件夹，在里边新建task.coffee，内容如下：

```
@Task = new Mongo.Collection('task')
```

Task collection就对应数据库里的一张表，根据需求，初步设计了下面几个字段：

```
name 任务名称
important 是否重要
urgent 是否紧急
complete 是否完成
create_time 创建时间
complete_time 完成时间
```

为了方便调试，要制造一些初始数据。在server文件夹下新建init_data.coffee

```
Meteor.startup(->
  if Task.find().count() is 0
    Task.insert
      name: "晚上和妹子去吃饭"
      important: true
      urgent: true
    Task.insert
      name: "翻译Meteor文档"
      important: true
      urgent: false
    Task.insert
      name: "看完《悲伤与理智》"
      important: false
      urgent: false
    Task.insert
      name: "给手机充话费"
      important: false
      urgent: true
)
```
解释下上面的代码：每次启动Meteor的时候，查询任务条数，如果条数为零，则插入四条初始数据。

之后打开数据库看看是否插入成功，打开一个新的命令行窗口，切换到项目根目录：启动mongo shell

```
meteor mongo
```

查询任务条数：

```
db.task.find().count()
1
```

明明插入了四条，为什么只查询到一条呢？因为在编码过程中，Meteor会监测源码的变化并自动重新运行，当刚写完第一条插入语句，Meteor可能就重新运行了，此时任务条数为1，后面的插入就不会再执行了。

接下来需要`control+c` 停止Meteor进程，重置并重启

```
meteor reset
meteor
```

再次到mongo shell中查询任务条数：

```
db.task.find().count()
4
```

## 将数据展示到前台界面

可以通过Helper给模板增加动态数据，在templates文件夹中新建home.coffee，内容如下：

```
Template.home.helpers
  taskList: ->
    Task.find()
```

修改home.html

```
<template name="home">
    {{#each taskList}}
        {{name}}
    {{/each}}
</template>
```

浏览器显示

	晚上和妹子去吃饭 翻译Meteor文档 看完《悲伤与理智》 给手机充话费

这里有一个问题需要思考：为什么客户端没有进行任何请求，数据就从服务端到了客户端？

这是因为：为了简化开发，每个新建的Meteor项目都会包含一个autopublish包，这个包会把所有集合里的文档自动发布到每一个连接上的客户端。为了能够精确地控制文档的发布，需要移除autopublish，然后手动进行发布与订阅

	meteor remove autopublish
	
此时浏览器中显示页面为空。接下来，在server目录下新建publish.coffee

```
Meteor.publish('tasks', ->
  Task.find()
)
```

在client目录下新建subscribe.coffee

```
Meteor.subscribe('tasks')
```

这时候浏览器中的页面数据就又都回来了。

## 页面样式调整

项目用到了yahoo的pure css框架:[点我下载](http://yui.yahooapis.com/pure/0.6.0/pure-min.css)

在client文件夹下新建style文件夹，将pure-min.css放进去，然后再新建一个todo.css 

将home.html 的布局修改为四象限

```
<template name="home">
    <div class="pure-g main-container">
        <div class="pure-u-1-2 task-list-wrapper task-list-wrapper-tl">
            <div class="task-list-title">
                重要&紧急
            </div>
            <ul class="task-list"></ul>
        </div>
        <div class="pure-u-1-2 task-list-wrapper  task-list-wrapper-tr">
            <div class="task-list-title">
                重要&不紧急
            </div>
            <ul class="task-list"></ul>
        </div>
        <div class="pure-u-1-2 task-list-wrapper  task-list-wrapper-bl">
            <div class="task-list-title">
                不重要&紧急
            </div>
            <ul class="task-list"></ul>
        </div>
        <div class="pure-u-1-2 task-list-wrapper  task-list-wrapper-br">
            <div class="task-list-title">
                不重要&不紧急
            </div>
            <ul class="task-list"></ul>
        </div>
    </div>
</template>
```

在todo.css 里增加样式

```
html, body {
    height: 100%;
}

.main-container {
    height: 100%;
}

.task-list-wrapper {
    height: 50%;
}

.task-list-wrapper-tl {
    background-color: #ffaeae;
}

.task-list-wrapper-tr {
    background-color: #56baec;
}

.task-list-wrapper-bl {
    background-color: #ffec94;
}

.task-list-wrapper-br {
    background-color: #b4d8e7;
}
```

接下来修改home模板的Helper

```
Template.home.helpers
  taskListTL: ->
    Task.find({important: true, urgent: true})
  taskListTR: ->
    Task.find({important: true, urgent: false})
  taskListBL: ->
    Task.find({important: false, urgent: true})
  taskListBR: ->
    Task.find({important: false, urgent: false})
```

修改home.html，将对应的Helper展示到对应的板块，例如：

```
<div class="pure-u-1-2 task-list-wrapper task-list-wrapper-tl">
            <div class="task-list-title">
                重要&紧急
            </div>
            <ul class="task-list">
                {{#each taskListTL}}
                    <li>{{name}}</li>
                {{/each}}
            </ul>
        </div>
```       

此时效果如下图：

![next 0.0.1 四象限](http://7xjw06.com1.z0.glb.clouddn.com/next 0.0.1 init.png)
## 新增任务

修改home.html，给每个版块增加一个form表单，用于新增任务

```
<div class="pure-u-1-2 task-list-wrapper task-list-wrapper-tl">
            <div class="task-list-title">
                重要&紧急
            </div>
            <form class="pure-form form-create-task">
                <input class="hidden" type="text" name="type" value="tl"/>
                <input class="pure-input-1" name="name" type="text"/>
                <button class="hidden" type="submit"></button>
            </form>
            <ul class="task-list">
                {{#each taskListTL}}
                    <li>{{name}}</li>
                {{/each}}
            </ul>
        </div>
```

表单里有一个隐藏的输入框，用于标识任务的类别，分别是：`tl`,`tr`,`bl`,`br`。
接下来，需要让模板监听表单的提交事件，并做处理。

在home.coffee 中增加：

```
Template.home.events
  'submit .form-create-task': (e)->
    e.preventDefault()
    $form = $(e.currentTarget)
    task =
      name: $form.find('input[name=name]').val()
      create_time: new Date()
      important: false
      urgent: false
    type = $form.find('input[name=type]').val()

    switch type
      when 'tl'
        task.important = true
        task.urgent = true
      when 'tr' then task.important = true
      when 'bl' then task.urgent = true
    console.log task
    Task.insert(task)
    $form.find('input[name=name]').val('')
```

首先阻止表单的默认提交，然后从表单中提取所需值，构建一个`task`对象，通过`Task.insert()`插入到数据库，然后清空表单。

在浏览器中测试没有问题，新增任务的功能就完成了。

## 修改任务

修改home.html，修改任务列表为：

```
<ul class="task-list">
                {{#each taskListTL}}
                    <li class="task-item" data-task-id="{{_id}}">
                        <div class="task-name" contenteditable="true">{{name}}</div>
                    </li>
                {{/each}}
            </ul>
```

修改home.coffee,监听`.task-name` 上的回车事件：

```
 'keypress .task-name': (e)->
    $el = $(e.currentTarget)
    taskId = $el.parent().attr('data-task-id')
    if e.keyCode is 13
      taskName = $el.text()
      Task.update({_id: taskId}, {$set: {name: taskName}})
      $el.blur()
      e.preventDefault()
```

浏览器中测试通过。

## 删除任务
修改home.html，修改任务列表为：

```
<ul class="task-list">
                {{#each taskListTL}}
                    <li class="task-item" data-task-id="{{_id}}">
                        <div class="task-name" contenteditable="true">{{name}}</div>
                        <span>&times;</span>
                    </li>
                {{/each}}
            </ul>
```

修改todo.css，调整样式：

```
.task-item {
    position: relative;
    line-height: 25px;
}

.delete-task {
    position: absolute;
    top: 0px;
    right: 10px;
    cursor: pointer;
}

.delete-task:hover {
    color: #dd3023;
}
```

修改home.coffee，绑定删除按钮的点击事件：

```
'click .delete-task': (e)->
    $el = $(e.currentTarget)
    taskId = $el.parent().attr('data-task-id')
    Task.remove({_id: taskId})
```

至此，Next 0.0.1 版本的需求就基本完成了。就是样式丑了些，先不要在意这些细节啦。后面再优化喽。