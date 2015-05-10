---
layout: post
category : lessons
title: "meteor collection"
tagline: "Supporting tagline"
tags : [meteor]
---


##Mongo.Collection

```
collection.allow(options)		server
允许用户直接从客户端修改collection，受到你定义的规则的限制。

Options
	insert,update,remove	Function
	如果允许返回true。
	fetch		Array of String
	可选项，可提升性能。限制从数据库获取的用于检查你的update 和remove 方法的字段数量。
	transform
	覆盖Collection上的transform。
```

当客户端在一个collection上调用insert，update，remove，collection的allow和deny会在服务端调用，来决定是否允许修改。如果至少有一个allow回调允许，并且没有deny回调拒绝，那么就允许操作。

这些检查只对客户端直接修改数据库起作用，例如：在一个事件处理中调用update。服务端代码是被信任的，不受allow和deny限制。包含那些通过Meteor.call 调用的method---它们需要自己进行权限检查，而不是依赖allow和deny。

你可以多次调用allow，每次调用可以随意组合insert，update，remove函数。如果允许操作，则返回true，否则返回false，或是什么都不返回。那样的话，Meteor会继续查找其它的allow规则。

可选的回调：

```
insert(userId,doc)
	doc will contain the _id field if one was explicitly set by the client, or if there is an active transform. You can use this to prevent users from specifying arbitrary _id fields.
	
update(userId,doc,fieldNames,modifier)
	doc 是数据库中的版本
	fieldNames 是客户端想要修改的doc的字段数组
	modifier 是Mongo原生的modifier，例如：{$set: {'name.first': "Alice"}, $inc: {score: 1}}
	只支持Mongo modifier。如果用户不使用$-modifiers ，尝试替换整个document，请求会直接被拒绝，而无需检查allow函数
	remove(userId,doc)
	
```
当调用update或是remove的时候，Meteor默认会从数据库中获取整个document。如果你希望只获取某些字段，使用fetch。

例如：

```
// Create a collection where users can only modify documents that
// they own. Ownership is tracked by an 'owner' field on each
// document. All documents must be owned by the user that created
// them and ownership can't be changed. Only a document's owner
// is allowed to delete it, and the 'locked' attribute can be
// set on a document to prevent its accidental deletion.

Posts = new Mongo.Collection("posts");

Posts.allow({
  insert: function (userId, doc) {
    // the user must be logged in, and the document must be owned by the user
    return (userId && doc.owner === userId);
  },
  update: function (userId, doc, fields, modifier) {
    // can only change your own documents
    return doc.owner === userId;
  },
  remove: function (userId, doc) {
    // can only remove your own documents
    return doc.owner === userId;
  },
  fetch: ['owner']
});

Posts.deny({
  update: function (userId, docs, fields, modifier) {
    // can't change owners
    return _.contains(fields, 'owner');
  },
  remove: function (userId, doc) {
    // can't remove locked documents
    return doc.locked;
  },
  fetch: ['locked'] // no need to fetch 'owner'
});
```

如果一个collection没有任何allow规则，那么客户端所有的写操作都会被拒绝，只能通过服务端代码进行修改。在这种情况下，你需要给每一种操作创建一个method。


##Mongo.Cursor
用find创建一个游标。用forEach、map、fetch获取游标里的文档。

##Mongo.ObjectID

##Selectors
##Modifiers
##Sort specifiers
##Field specifiers




