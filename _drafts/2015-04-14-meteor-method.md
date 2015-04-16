---
layout: post
category : lessons
title: "meteor method"
tagline: "Supporting tagline"
tags : [meteor]
---

Methods 是客户端可以调用的远程函数

```
Meteor.method(methods)		Anywhere
	定义远程函数，客户端通过网络调用。
	Arguments
		method		Object
		字典，键是方法名，值为函数
```

例如：

```
Meteor.methods({
  foo: function (arg1, arg2) {
    check(arg1, String);
    check(arg2, [Number]);

    // .. do stuff ..

    if (/* you want to throw an error */) {
      throw new Meteor.Error("pants-not-found", "Can't find my pants");
    }

    return "some return value";
  },

  bar: function () {
    // .. do other stuff ..
    return "baz";
  }
});
```

method 应该返回EJSON-able 值或是抛出异常。在method调用里面，this指向一个method调用对象，它提供下面的属性和方法：

- isSimulation:布尔值，如果本调用是一个stub，则为true
- unblock:调用unblock时，允许当前客户端调用的下一个method开始运行
- userId：当前用户的ID
- setUserId：把当前客户端关联到一个用户
- connnection：

在客户端调用method，会定义与服务端method关联的stub，且名字相同。如果你不想也可以不定义stub。这样的话，method调用就像远程程序调用，必须等到服务端返回结果。

如果定义了stub，当客户端调用服务端method时，会并行运行stub。在客户端，stub的返回值被忽略。stub运行是为了复现它们的side-effects:为了模拟服务端会出现的结果，但是无需等待网络传输。如果stub抛出异常，会被记录到console

其实，你一直都在使用method，因为数据库的修改方法(insert,update,remove)都是以method方式实现。当你在客户端调用这些方法时，实际上是在调用它们的stub版，来更新本地缓存，发送相同的请求到服务端。当服务端返回响应时，客户端用实际发生在服务端的变化更新本地缓存。

由于method通常期望特定的参数类型，使用check来保证参数的类型和结构正确。

如果客户端调用method之后，在收到响应之前，断开了与服务端的链接，当重新连接之后，它会重新调用对应method。这意味着，客户端有可能多次调用同一个method。如果这对你的method是一个问题，可以考虑附加一个unique ID给每一个method调用,并在服务端检查该ID是否已经调用过。

```
this.userId		Anywhere
	调用该method的用户的ID，如果没有用户登录则为null
```

用户的ID是任意的字符串--通常是数据库中用户记录的ID。你可以用setUserId函数设置。如果你使用了Meteor 账号系统，会自动为你处理。

```
this.setUserId(userId)
	设置登录的用户
	Arguments
		userId		String or null
		
```













