---
layout: post
category : lessons
title: "meteor 账户系统"
tagline: "Supporting tagline"
tags : [meteor]
---
##Accounts

最基本的账户系统在account-base 包里。

```
Meteor.user()		anywhere 除了publish function
	获取当前用户记录，如果没有登录，则为null。是一个reactive 数据源。
```

从Meteor.users collection里检索当前用户的记录。

在客户端，结果是服务端发布的数据的子集(一些域客户端不可见)。默认情况下，服务端发布username，email，和profile。

```
Meteor.userId()		anywhere 除了publish function
	获取当前用户ID，如果没有登录则为null。是一个reactive 数据源

```


```
Meteor.users		anywhere
包含用户documents的Mongo.Collection
```
每一个注册用户都有一条记录。例如：


```
{
  _id: "bbca5d6a-2156-41c4-89da-0329e8c99a4f",  // Meteor.userId()
  username: "cool_kid_13", // unique name
  emails: [
    // each email address can only belong to one user.
    { address: "cool@example.com", verified: true },
    { address: "another@different.com", verified: false }
  ],
  createdAt: Wed Aug 21 2013 15:16:52 GMT-0700 (PDT),
  profile: {
    // The profile is writable by the user by default.
    name: "Joe Schmoe"
  },
  services: {
    facebook: {
      id: "709050", // facebook id
      accessToken: "AAACCgdX7G2...AbV9AZDZD"
    },
    resume: {
      loginTokens: [
        { token: "97e8c205-c7e4-47c9-9bea-8e2ccc0694cd",
          when: 1349761684048 }
      ]
    }
  }
}
```

你可以在里面保存任何数据。但是有几个域，Meteor会特殊对待：

- `username`:唯一的字符串，可以区别用户
- `email`：对象数组，key 是address 和verified。每个邮箱只能属于一个用户。verified是一个布尔值，
- `createAt`：当前用户记录创建时间
- `profile`：对象，用户可以用任何数据创建和更新。不要把你不想用户修改的数据保存到profile，除非Meteor.users collection上有deny 规则
- `services`：对象，包含特定登录服务使用的数据。例如：reset 域包含token，忘记密码链接会用到。resume 域包含的token会用于session之间保持登录

和其他collection一样，在服务端你可以获取所有document，但是只有那些明确发布的域，可以在客户端获取到。

默认情况下，当前用户的username，emails,profile会发布到客户端。你可以发布其它的数据：


```
Meteor.publish("userData", function () {
  if (this.userId) {
    return Meteor.users.find({_id: this.userId},
                             {fields: {'other': 1, 'things': 1}});
  } else {
    this.ready();
  }
});

// client
Meteor.subscribe("userData");
```

如果安装了autopublish包，所有用户的所有信息都会发布到所有客户端，包括service 域里的信息。

默认情况下，在Accounts.createUser中，用户可以声明自己的profile域，还可以通过Meteor.users.update 修改它。用Meteor.users.allow来允许用户编辑其它域。禁止用户做任何修改：

```
Meteor.users.deny({update: function () { return true; }});
```


```
Meteor.loggingIn()		Client
	如果正在调用一个登录method(例如：Meteor.loginWithPassword, Meteor.loginWithFacebook, or Accounts.createUser)，返回true。reactive数据源。
```
account-ui 包利用它来显示登录等待动画。

```
Meteor.logout([callback])		Client
注销当前用户。

参数
	callback	函数
	可选callback。成功时，无参数调用，失败时，带有一个Error参数
```

```
Meteor.logoutOtherClients([callback])	    client
注销以当前用户身份登陆的其它客户端，除了当前客户端。

参数
	callback	函数
	可选callback。	成功时，无参数调用，失败时，带有一个Error参数				
```

例如：在客户端调用时，当前浏览器的连接保持登录，其它浏览器的DDP客户端，以该用户登录的会被注销。


```
Meteor.loginWithPassword(user,password,[callback])      client
用账号登录

参数
	user 对象或字符串
	username 或email ；或者是一个只有一个key(email,username,id)的对象
	password	字符串
	密码
	callback	函数
	可选callback。	成功时，无参数调用，失败时，带有一个Error参数
	
```

```
Meteor.logWith<ExternalService>([options],[callback])   client

```

```
{{currentUser}}
调用Meteor.user()。用{{#if currentUser}}检查是否登录
```

```
{{loggingIn}}
调用Meteor.loggingIn()
```

```
Accounts.config(options)
设置全局账户配置

Options
	sendVerificationEmail	布尔值
	用邮箱注册的新用户会收到验证邮件
	forbidClientAccountCreation	布尔值
	在客户端调用createUser 会被拒绝
	restrictCreationByEmailDomain
	loginExpirationInDays
	默认90天，设为null 即可永不过期
	oauthSecretKey
	
```

```
Accounts.ui.config(option)
配置{{>loginButtons}}的行为

Options
	
```
 

```
Accounts.validateNewUser(func)	server
创建新用户时设置限制

参数
	func	函数
	当创建新用户时被调用。新用户对象作为参数，如果允许就返回true，返回false 会终止创建。
```
可以多次调用。


```
Accounts.onCreateUser(func)		server
自定义新用户创建

参数
	func	函数
	创建新用户时被调用。返回新用户对象或是抛出Error
```

```
Accounts.validateLoginAtempt(func)		server
验证登录企图
```

```
Accounts.onLogin(func)		anywhere
注册回调函数，当登录成功时调用
```


```
Accounts.onLoginFailure(func)		anywhere
注册回调函数，当登录失败时调用
```

##Passwords

```
Accounts.createUser(options.[callback])	   anywhere
创建一个新用户
```








