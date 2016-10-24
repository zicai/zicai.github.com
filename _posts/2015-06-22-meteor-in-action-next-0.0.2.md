---
layout: post
category : lessons
title: "Meteor 项目实战 -- Next 0.0.2"
tagline: "Supporting tagline"
tags : [meteor]
---

接上一篇:[Meteor项目实战 -- Next 0.0.1](http://zicai.github.io/lessons/2015/06/22/meteor-in-action-next-0.0.1/)

> Get things done , and do Next


Next 0.0.2版的目标是账户系统，并把任务与用户关联起来。

首先增加登录所需路由，修改router.coffee，增加如下代码：

```
Router.route('/login', ->
  @render("Login")
)

```

增加对应模板，在client/templates/ 下新建login.html，内容如下：

```
<template name="Login">
    welcome login
</template>
```

Meteor提供了方便好用的账户系统。账户系统的基本功能位于`accounts-base`包，但通常都会选择直接引入--登录服务提供包，例如：

- `accounts-password`
- `accounts-facebook`
- `accounts-github`
- `accounts-google`
- `accounts-meetup`
- `accounts-twitter`
- `accounts-weibo`

引入这些包时会自动引入`accounts-base`包。

使用第三方登录不仅能让用户省去繁琐的注册流程，还能大大简化开发。考虑到国情，本来打算使用`accounts-weibo`，但是体验了之后发现，微博的接入流程还需要审核一堆材料，所以我们这里选择引入`accounts-github`。

```
meteor add accounts-github
```

观察控制台的输出，我们发现自动添加的依赖包有：

```
accounts-base          added, version 1.2.0
accounts-github        added, version 1.0.4
accounts-oauth         added, version 1.1.5
github                 added, version 1.1.3
localstorage           added, version 1.0.3
oauth                  added, version 1.1.4
oauth2                 added, version 1.1.3
service-configuration  added, version 1.0.4
```

然后在 mongo shell 中 `show collections;` 发现同时多了两个集合：

```
meteor_accounts_loginServiceConfiguration
meteor_oauth_pendingCredentials
```

从集合的名字推断：大概是用来保存第三方登录配置和凭证的。

使用第三方登录服务之前，都需要进行一些配置。最简单的方式就是引入另一个包 `accounts-ui`,它给每一个登录服务提供一个配置向导。

```
meteor add accounts-ui
```

然后修改login.html，为

```
<template name="Login">
    {{> loginButtons}}
</template>
```

`{{> loginButtons}}`是`accounts-ui`包提供的登录UI。

访问登录页，会出现一个configure github login的按钮，点击它，会出现一个向导，按照步骤提示进行配置，然后点击save configuration。

![configure github login][1]

这时候按钮变成了sign in with github。

在mongo shell中执行：

```
db.meteor_accounts_loginServiceConfiguration.find()
```

可以看到刚才的配置就保存到了这个集合中。

接下来，我们点击sign in with github。就会跳到github的授权页面，点击同意，就会跳转回登录页，按钮变成了sign out，说明已经成功登录

利用第三方登录服务成功登录后，Meteor会自动在users集合中新建对应的用户，到mongo shell中查询，

```
db.users.find().pretty()
``` 

输出结果类似：

![查询用户][2]

到目前为止一切顺利。接下来调整一些细节，当未登录用户访问时，自动跳转到登录页。修改router.coffee，增加下面代码：

```
Router.onBeforeAction(->
  if !Meteor.userId()
    this.redirect('/login')
  this.next()
)
```

现在我们有了账户系统，然后就要把任务和用户关联起来。

首先，新建任务时，同时保存用户的`_id`字段，修改home.coffee

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
      user_id: Meteor.userId() // 保存用户_id字段
      
      ....后面代码省略
```

最后，修改发布，只发布当前用户的任务，修改publish.coffee,

```
Meteor.publish('tasks', ->
  Task.find({user_id: @userId})
)
```
Done！ ^_^

  [1]: http://segmentfault.com/img/bVmPOD
  [2]: http://segmentfault.com/img/bVmPOz






























