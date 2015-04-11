---
layout: post
category : lessons
title: "meteor tutorial"
tagline: "Supporting tagline"
tags : [meteor]
---



![meteor logo](http://docs.meteor.com/logo.png)

> Build apps that are a delight to use, faster than you ever thought
> possible

##Meteor 是什么？
Meteor 是一个构建在 Node.js 之上的平台，用javascript开发web 和mobile apps。完全开源。

##它的快体现在哪里？
###对用户：实时响应，无需等待
- 客户端路由 [Iron.Router](http://iron-meteor.github.io/iron-router/)
- 延迟补偿（Latency Compensation）

###对开发者：缩短开发时间
- Radically less code : reactive programming model
- One language everywhere : javascript
- Unified package system（browser，server，mobile）
- Hot deploys : [meteor up](https://github.com/arunoda/meteor-up)
- Database Everywhere

##快速开始
[Tutorial](https://www.meteor.com/install)
模板
collection
部署
移动版
session
账户系统
method
发布订阅

##packages
Meteor 所有的功能都以包的形式实现。核心包、第三方包。

    meteor list
    meteor add package-name
    meteor remove package-name
    
当你添加或者删除包得时候，程序会自动重启。包的依赖记录在`.meteor/packages`。

可以通过`meteor search` 查找包。也可以通过官方包服务器[atmospherejs](https://atmospherejs.com/)

通过`meteor show package-name`查看包信息。

包名里带`:`的，说明它由个人或组织维护。不带`:`，说明是官方维护的，作为Meteor框架的一部分。

常用包：

- coffeescipt
- accounts-ui
- iron:router

##文件结构
一个Meteor App会包含客户端javascript(运行在浏览器或PhoneGap mobile app)，服务端javascript(运行在Node.js)，HTML template，CSS，静态文件。

###默认文件加载
Meteor 会自动加载所有文件，所以无需再用`<script>`和`<link>`来引用外部脚本文件和样式文件。

如果一个javascript文件不在下面的特殊文件中，那么它会被发送到客户端和服务端。

###特殊文件夹
- client 只在客户端使用，用于放置HTML、CSS和UI相关的javascript。
- server 只在服务端使用，不会被发送到客户端。用于放置敏感的逻辑和数据，客户端不可见。Meteor收集除了client、public、private文件夹中的所有js文件，然后把它们加载到node.js server实例中。
- public(top-level)  放置图片、字体等资源文件。
- private(top-level) 只能在服务端通过Assets API 获取
- tests 
- client/compatibility 

###文件加载顺序
1. HTML 模板文件总是最先加载
2. 以`main.`开头的文件最后加载
3. `lib/`文件夹里的优先加载
4. 目录层次更深的优先加载
5. 按照整个路径的字母顺序加载

例如：

    nav.html
    main.html
    client/lib/methods.js
    client/lib/styles.js
    lib/feature/styles.js
    lib/collections.js
    client/feature-y.js
    feature-x.js
    client/main.js

##数据从哪来？---Collection
Meteor把数据保存在collection里。collection里保存的javascript对象叫做document。

声明一个collection

```
new Mongo.Collection(name,[option])
```

注意：需要将将collection声明为全局变量，这样服务端和客户端才能用相同的API操作相同的collection。

声明了collection之后，就可以对其进行操作了

```
collection.findOne()
collection.find()
collection.insert()
collection.update()
collection.remove()
```

###安全性
默认情况下，新建的Meteor app里，客户端是可以直接调用insert，update，remove 。这是因为为了简化开发，meteor create 的时候默认包含了一个insecure 包。

为了安全考虑，我们需要先移除insecure包，

```
meteor remove insecure
```
默认情况下，移除`insecure`之后，客户端所有的操作都会被拒绝。所以，我们需要添加一些`allow`规则：

```
// In a file loaded on the server (ignored on the client)
Posts.allow({
  insert: function (userId, post) {
    // can only create posts where you are the author
    return post.createdBy === userId;
  },
  remove: function (userId, post) {
    // can only delete your own posts
    return post.createdBy === userId;
  }
  // since there is no update field, all updates
  // are automatically denied
});
```

allow 方法可以接受三个回调函数，分别对应insert,remove,update。每个回调函数的第一个参数是登录用户的_id，其它的参数如下：

```
insert(userId, document)
update(userId, document, fieldNames, modifier)
remove(userId, document)
```
如果返回true，说明允许，否则拒绝。

deny 方法用来重写allow 规则。例如：

```
// In a file loaded on the server (ignored on the client)
Posts.deny({
  insert: function (userId, post) {
    // Don't allow posts with a certain title
    return post.title === "First!";
  }
});
```

注意：allow和deny规则只对客户端代码生效，对服务端和methods里的代码不起作用。

##数据展示到哪里？---模板
模板就是包含动态数据的HTML片段。可以通过javascript给模板插入数据和监听事件。

###定义模板
模板定义在.html文件中，可以放在任何文件夹，除了server，public，private。

每一个.html 文件中可以包含任意数量的顶级元素：`<head>, <body>,  <template>`。例如：

```
<!-- add code to the <head> of the page -->
<head>
  <title>My website!</title>
</head>

<!-- add code to the <body> of the page -->
<body>
  <h1>Hello!</h1>
  {{> welcomePage}}
</body>

<!-- define a template called welcomePage -->
<template name="welcomePage">
  <p>Welcome to my website!</p>
</template>
```

Meteor 通过[Spacebars](https://github.com/meteor/meteor/tree/devel/packages/spacebars) 给HTML添加功能。例如：可以通过`{{>template-name}}`来插入模板，通过`{{variable}} `来显示数据，以及逻辑控制标签`{{#each}}{{/each}}`,`{{#if}}{{/if}}`

###给模板定义数据
模板的数据有一部分来自controller(后面再说)，还有一部分来自helper。
helper 可以是简单的一个值，也可以是个函数，还可以带参数。
```
Template.myTemplate.helpers(helpers)
```

例如：

```
Template.nametag.helpers({
  name: "Ben Bitdiddle"
});
```
然后就可以在模板里显示：

```
<template name="nametag">
  <p>My name is {{name}}.</p>
</template>
```

上面的方法是给特定模板添加helper，可以通过`Template.registerHelper`来注册一个所有模板都可以获取的helper。

###监听模板事件

```
Template.myTemplate.events(eventMap)
```
假设有下面的模板：

```
<template name="example">
  {{#with myHelper}}
    <button class="my-button">My button</button>
    <form>
      <input type="text" name="myInput" />
      <input type="submit" value="Submit Form" />
    </form>
  {{/with}}
</template>
```

给模板注册两个事件：

```
Template.example.events({
  "click .my-button": function (event, template) {
    alert("My button was clicked!");
  },
  "submit form": function (event, template) {
    var inputValue = event.target.myInput.value;
    var helperValue = this;
    alert(inputValue, helperValue);
  }
});
```

事件描述符作为key，事件处理函数作为value。事件处理函数会收到两个参数，一个是事件对象，一个是当前模板实例。事件处理函数还可以通过`this` 获取当前的数据上下文(相当于[template.data](http://docs.meteor.com/#/full/template_data))。

key的前半部分(空格之前)是要捕获的事件名称。几乎支持所有的DOM事件。常见的有：`click`, `mousedown`, `mouseup`, `mouseenter`, `mouseleave`, `keydown`, `keyup`, `keypress`, `focus`, `blur`, 和 `change`。

key的后半部分是CSS选择器，指明要监听的元素。几乎支持所有jQuery选择器。

###onRendered

```
Template.myTemplate.onRendered(callback)
```

类似于jQuery里的

```
$(document).ready(callback)
```

###模板实例
模板实例可以用来获取模板中的HTML元素，还可以给模板实例添加属性，用于在reactive update之间保持数据。

模板实例可以通过下面方式获取：

- `onCreated`,`onRendered`,`onDestroyed` 回调函数里`this`的值
- 事件处理函数的第二个参数
- 在helper里通过`Template.instance()`

模板实例有很多方法和属性，例如,选取DOM元素：

```
template.findAll(selector)
template.find(selector)
//还可以用
template.$()
```

获取数据上下文

```
tempalte.data
```

##数据如何到达模板？---发布和订阅
![推送数据库子集到客户端](https://s3.amazonaws.com/discovermeteor/diagrams/client-server@2x.png)

Database everywhere。简单地说，Meteor会把服务端的一部分数据复制到客户端。

这样做的结果：

1. 服务器不再发送 HTML 代码到客户端，而是发送真实的原始数据，让客户端决定如何渲染数据。
2. 用户不必等待服务器传回数据，而是立即访问甚至修改数据（延迟补偿 latency compensation）

新创建的Meteor App 都会包含一个autopublish的包，它会自动的把所有的documents发布到每一个连接上的客户端。

出于安全性等考虑，我们不能把服务端数据全部都推到客户端。通过Meteor的*发布*功能来精确控制哪些数据子集是要推送的。

首先移除autopublish包

```
meteor remove autopublish
```

假设数据库中存了一些文章，如下：

![数据库中所有文章](https://s3.amazonaws.com/discovermeteor/diagrams/collections-1@2x.png)

其中，有一些文章是有特殊标记的，不允许客户端看到：

![特殊标记的文章](https://s3.amazonaws.com/discovermeteor/diagrams/collections-2@2x.png)

在服务端，就可以这样写：

```
Meteor.publish('posts', function() {
  return Posts.find({flagged: false}); 
});
```

在客户端订阅：

```
Meteor.subscribe('posts');
```

更进一步，假设我们正在浏览Bob Smith的个人主页，这里只显示它自己的文章：

![](https://s3.amazonaws.com/discovermeteor/diagrams/collections-3@2x.png)

修改发布：

```
Meteor.publish('posts', function(author) {
  return Posts.find({flagged: false, author: author});
});
```

修改订阅：

```
Meteor.subscribe('posts', 'bob-smith');
```

##路由
路由就是配置。

###配置写在哪？
Iron.Router 按下面顺序查询路由配置

1. RouteController

	```
	myController=RouteController.extend({
		layoutTemplate:'layout'
	})
	```
2. Route
	
	```
	Router.route('path',路由函数,配置对象)
	```
3. Router

	```
	Router.configure({
		layoutTemplate:'layout',
		template:'temp'
	})
	```

###有哪些配置项？

- name
- path
- controller
- template
- layoutTemplate
- yieldRegions

	```
	yieldRegions:{
		'mySide':{to:'aside'},
		'myFooter':{to:'footer'}
	}
	```
- waitOn
- data
- onRun
- onRerun
- onBeforeAction
- onAfterAction
- onStop
- action
- where

###路由执行流程


##保持状态---session
session是存在于客户端的全局对象。可以用它来保存任意的键值对。

##账号系统


##它的支撑
- Blaze
- Tracker
- DDP
- LiveQuery
- Full Stack Database Driver
- isobuild

##它的核心
reactive programing
###它用来解决什么问题
一个很常见的需求：监测某一些值，当值发生变化时，执行某些操作。

常见的解决思路：
- poll and diff
- events
- bindings

另外一种就是reactive programing。思考下电子表格。

reactive programing非常适合开发用户界面。


[谈谈UI架构设计的演化](http://www.cnblogs.com/winter-cn/p/4285171.html)
[Web前端开发：为何选择MVVM而非MVC](http://www.cnblogs.com/winter-cn/archive/2012/09/16/2687184.html)



