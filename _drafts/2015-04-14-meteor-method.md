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

在客户端调用Meteor.methods，会定义与服务端method关联的stub，且名字相同。如果你不想也可以不定义stub。这样的话，method调用就像远程程序调用，必须等到服务端返回结果。

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

```
this.isSimulation		anywhere
布尔值，如果调用的是stub,则为true
```

```
this.unblock()		server
允许随后的来自该客户端的请求在一个新fiber中开始运行。
```

在服务端，来自同一个客户端的方法调用，一次执行一个。第N+1个方法调用必须等待第N个调用返回。但是可以通过调用this.unblock来改变。允许第N+1个方法调用在一个新fiber中运行。

```
this.connection		server
接受方法调用的connection。如果方法调用没有关联到一个connection，则为null，例如服务端初始方法调用。
```

```
new Meteor.Error(error,[reason],[detail])  anywhere

参数
	error		string
	用来标识这种错误的字符代码。方法调用者可以用字符代码来确定执行何种操作。例如：
	// on the server, pick a code unique to this error
	// the reason field should be a useful debug message
	throw new Meteor.Error("logged-out", 
  		"The user must be logged in to post a comment.");

	// on the client
	Meteor.call("methodName", function (error) {
  	// identify the error
  	if (error.error === "logged-out") {
    	// show a nice error message
    	Session.set("errorMessage", "Please log in to post a comment.");
  	}
	});
	历史遗留问题：一些内置的Meteor函数，例如check,error字段用的是数字，而不是字符串
	reason		string
	可选的。简短易读的错误概要，例如：'not found'
	details 	string
	可选的。错误的详细信息，例如堆栈。
```

如果你想在一个方法里返回一个错误，那么就抛出异常。方法可以抛出任何异常。但是Meteor.Error 是唯一一种服务端会发送到客户端的error。如果抛出了其它异常，会映射到一个净化版。特别的，如果抛出错误的sanitizedError字段指定为Meteor.Error,那么错误会被发送到客户端。否则，如果没有sanitized 版本，客户端会收到Meteor.Error(500,'Internal server error')


```
Meteor.call(name,[arg1,arg2...],[asyncCallback])    anywhere
调用一个method，传递一些参数

参数：
	name		string
	arg1,arg2..		EJSON-able 对象
	可选method参数
	asyncCallback  	Function
	可选的回调函数，








If you include a callback function as the last argument (which can't be an argument to the method, since functions aren't serializable), the method will run asynchronously: it will return nothing in particular and will not throw an exception. When the method is complete (which may or may not happen before Meteor.call returns), the callback will be called with two arguments: error and result. If an error was thrown, then error will be the exception object. Otherwise, error will be undefined and the return value (possibly undefined) will be in result.