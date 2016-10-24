---
layout: post
category : lessons
title: "Meteor 发布与订阅"
tagline: "Supporting tagline"
tags : [Meteor]
---

Meteor服务端可以通过`Meteor.publish`发布一组文档，客户端可以通过`Meteor.subscribe` 订阅这些文档。客户端订阅的所有的文档都可以通过客户端的collections的`find`方法查询到。

默认情况下，所有新创建的Meteor的app都包含autopublish包，它会自动发布所有文档到每一个客户端。 要想更细粒度的控制每个客户端收到的文档,首先移除autopublish

```
$ meteor remove autopublish
```

现在你就可以通过`Meteor.publish`和`Meteor.subscribe`来控制服务端发送哪些文档到客户端。

```
Meteor.publish(name, func) 				定义在Server
	发布一个记录集.

	Arguments
		name 	String
		记录集的名字。如果为 null, 则记录集没有名字，并且该记录集会自动发布到所有已连接上的客户端。

		func 	Function
		每次有客户端订阅时，调用该函数。在函数中， this 代表publish handler对象, 
		下面有介绍。如果客户端传递了参数给subscribe, 该函数会以相同参数被调用。
```

要发布数据到客户端，在服务端调用`Meteor.publish`，带上两个参数：记录集的名字，和一个发布函数，每次有客户端订阅这个记录集的时候，这个函数就会被调用。

发布函数可以返回一个`Collection.Cursor`，Meteor会把游标对应的文档发布到每一个订阅的客户端。

```
// Publish the logged in user's posts
Meteor.publish("posts", function () {
  return Posts.find({ createdBy: this.userId });
});
```

还可以返回一个`Collection.Cursor`的数组：

```
// Publish a single post and its comments
Meteor.publish("postAndComments", function (postId) {
  // Check argument
  check(postId, String);

  return [
    Posts.find({ _id: postId }),
    Comments.find({ postId: roomId })
  ];
});
```

注意：如果返回的是游标数组，在当前版本中必须是不同的collection，将来可能取消这一限制。

另一种方式：发布函数可以通过调用`added,changed,removed`方法直接控制发布的记录集。这些方法由发布函数中的`this`提供。

如果一个发布函数没有返回游标或者游标数组，则会假定你使用了底层的`added/changed/removed` 接口，同时必须在记录集初始化完成时调用`ready`方法。

例如：

```
// server: publish the current size of a collection
Meteor.publish("counts-by-room", function (roomId) {
  var self = this;
  check(roomId, String);
  var count = 0;
  var initializing = true;

  // observeChanges only returns after the initial `added` callbacks
  // have run. Until then, we don't want to send a lot of
  // `self.changed()` messages - hence tracking the
  // `initializing` state.
  var handle = Messages.find({roomId: roomId}).observeChanges({
    added: function (id) {
      count++;
      if (!initializing)
        self.changed("counts", roomId, {count: count});
    },
    removed: function (id) {
      count--;
      self.changed("counts", roomId, {count: count});
    }
    // don't care about changed
  });

  // Instead, we'll send one `self.added()` message right after
  // observeChanges has returned, and mark the subscription as
  // ready.
  initializing = false;
  self.added("counts", roomId, {count: count});
  self.ready();

  // Stop observing the cursor when client unsubs.
  // Stopping a subscription automatically takes
  // care of sending the client any removed messages.
  self.onStop(function () {
    handle.stop();
  });
});

// client: declare collection to hold count object
Counts = new Mongo.Collection("counts");

// client: subscribe to the count for the current room
Tracker.autorun(function () {
  Meteor.subscribe("counts-by-room", Session.get("roomId"));
});

// client: use the new collection
console.log("Current room has " +
            Counts.findOne(Session.get("roomId")).count +
            " messages.");

// server: sometimes publish a query, sometimes publish nothing
Meteor.publish("secretData", function () {
  if (this.userId === 'superuser') {
    return SecretData.find();
  } else {
    // Declare that no data is being published. If you leave this line
    // out, Meteor will never consider the subscription ready because
    // it thinks you're using the added/changed/removed interface where
    // you have to explicitly call this.ready().
    return [];
  }
});

```
发布函数通常都期望特定的参数类型，使用`check`来保证参数的类型和结构。

```
this.userId			server
在发布函数内部获取登录用户的id,如果没有登录则为null
```
这是一个常量。但是，如果登录用户发生变化，发布函数会重新运行。

```
this.add(collection,id,fields)			server
在发布函数内部调用，通知订阅者：一个文档被添加到记录集中

参数
	collection	string
	包含新文档的collection name。
	id		string
	新文档的ID
	fields		Object
	新文档包含的字段。如果有_id字段，会被忽略
	
```

```
this.changed(collection,id,fields)		server
在发布函数内部调用，通知订阅者：记录集里的一个文档发生了变化。

参数
	collection	string
	id		string
	fields		Object
	发生变化的字段，包含新的值。如果一个字段没有出现在fields,表示没有变化；如果出现在fields中，值为undefined，会从文档中移除。如果有_id字段，会被忽略
```

```
this.removed(collection,id)			server
在发布函数内部调用，通知订阅者：一个文档从记录集中移除

参数
	collection		string
	id		string
```

```
this.ready()			server
在发布函数内部调用。通知订阅者：一个最初的，记录集的完整快照已经发送。如果Meteor.subscribe有onReady回调的话，这会触发客户端的onReady回调函数。
```

```
this.onStop(func)			server
在发布函数内部调用。注册一个回调函数，当订阅停止时调用。

参数
	func		Function
	回调函数
```

如果你在publish handler中调用了`observe`或是`observeChanges`,需要在`onStop`回调函数中停止`observe`。

```
this.error(error)		server
在发布函数中调用。停止客户端的订阅，如果Meteor.subscribe有onStop的话，则触发客户端的onStop回调函数。如果error不是一个Meteor.Error，会被sanitized.

参数
	error 		Error
	传给客户端的error
```


```
this.stop()			server
在发布函数中调用。停止客户端的订阅，触发客户端的onStop回调，不带error
```

```
this.connection			server
在发布函数中获取订阅的connection
```

```
Meteor.subscribe(name, [arg1, arg2...], [callbacks]) 	定义在Client
	订阅一个记录集，返回一个handler，handler提供stop()和ready()方法。

	Arguments
		name String
		订阅的名字，要和服务端的publish()的名字一致

		arg1, arg2... Any
		可选的参数，会传递给服务端的publish函数。

		callbacks Function or Object
		可选。可以包含onStop和onReady回调函数。如果发生错误，会作为参数传递给 onStop. 如果callbacks是个函数形式而不是对象, 函数会被解释为onReady 。
```
当你订阅一个记录集时，它通知服务端发送一些记录到客户端。客户端把这些记录保存到本地的Minimongo collection，用和publish handler中`added，changed,removed`回调函数里collection参数一样的名字。Meteor会queue incoming 记录，直到你在客户端以相同的名字声明Mongo.Collection。

```
// okay to subscribe (and possibly receive data) before declaring
// the client collection that will hold it.  assume "allplayers"
// publishes data from server's "players" collection.
Meteor.subscribe("allplayers");
...
// client queues incoming players records until ...
...
Players = new Mongo.Collection("players");
```

当服务端把所有初始数据发送给订阅后，`onReady`回调函数就会被调用，不带参数。当订阅因为任何原因被终止后`onStop`回调函数就会被调用，如果订阅是由于服务端错误导致的失败的话，它会收到一个`Meteor.Error`

`Meteor.subscribe`返回一个订阅处理器，是一个对象，有下面的属性：


```
	stop()
	取消订阅。通常结果是服务端指示客户端从客户端缓存中移除订阅数据。

	ready()
	如果服务端标记订阅为ready，则返回true。一个reactive数据源。
	
	subscrptionId
	这个handler对应的订阅的ID。当你在Tracker.autorun中执行meteor.subscribe ,你获得的handler始终有相同的subscriptionId.可以利用这一点来去掉重复的subscription handler。
```
如果你在一个reactive computation中调用`meteor.subscribe`，例如`Tracker.autorun`，当computation is validated或是停止时，订阅会自动被取消；无需在autorun中调用`stop`。而且，当autorun重新运行时，对于订阅相同集合的订阅(相同的name 和参数)，Meteor会很聪明的跳过 取消订阅/再订阅 这一开销很大的行为。例如：

```
Tracker.autorun(function () {
  Meteor.subscribe("chat", {room: Session.get("current-room")});
  Meteor.subscribe("privateMessages");
});

```
如果多个订阅返回了冲突的字段值，那么客户端会武断的选择一个值。

















