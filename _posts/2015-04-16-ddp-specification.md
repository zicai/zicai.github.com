---
layout: post
category : lessons
title: "DDP 规范"
tagline: "Supporting tagline"
tags : [DDP]
---

DDP是一个客户端和服务端之间的通信协议，支持两种操作：

- 客户端到服务端的远程调用
- 客户端订阅一些document，当document发生变化时服务端通知客户端。

这篇文档是版本1 DDP的说明。是对协议一个粗略的描述，而不是完整定义。

## General Message Structure:

DDP使用SockJS或是WebSockets作为底层的消息通道。(For
now, you connect via SockJS at the URL `/sockjs` and via WebSockets at the URL
`/websocket`. The latter is likely to change to be the main app URL specifying a
WebSocket subprotocol.)


DDP消息是JSON对象，再加上一些指定的字段，称为EJSON。每一条消息都有一个`msg`字段，用来指定消息的类型，其它的字段依赖于消息类型。

客户端和服务端必须忽略消息里未知的的字段。Future
minor revisions of DDP might add extra fields without changing the DDP version;
the client must therefore silently ignore unknown fields.  However, the 客户端
must not send extra fields other than those documented in the DDP protocol, in
case these extra fields have meaning to future servers.  On the server, all
field changes must be optional/ignorable for compatability with older clients;
otherwise a new protocol version would be required.


## Establishing a DDP Connection:

### Messages:

 * `connect` (客户端 -> 服务端)
   - `session`: string (如果尝试重连已存在的DDP会话)
   - `version`: string (提议的协议版本)
   - `support`: array of strings (客户端支持的协议版本,按偏好排序)
 * `connected` (服务端->客户端)
   - `session`: string (DDP 会话的标示符)
 * `failed` (服务端->客户端)
   - `version`: string (建议使用的协议版本)

### Procedure:

服务端可能会发送一个初始化消息，是一个没有`msg`键的JSON对象。如果客户端收到这条消息，忽略即可。客户端无需等待这条消息。（这条消息之前是用来实现Meteor的hot code reload;现在用来强制旧客户端更新）

 * 客户端发送一条`connect`消息
 * 如果服务端愿意用`connet`消息里`version`指定的协议版本回应，它就回复一条`connected`消息。
 * 否则服务端发送一条`failed`消息，带上自己期望的DDP版本，由`connect`消息里`support`字段提示的，然后关闭底层的通道。
 * 客户端之后可以尝试以不同的DDP版本重连。通过在一个新连接上发送另一条`connect`消息。客户端可以假设服务端支持该版本，发送`connect`之后继续发送更多的消息。如果服务端不支持该版本，忽略这些消息。

客户端connect消息里的support字段里的DDP版本，按照客户端的偏好进行排序，最偏好的排在首位。如果，按照排序，客户端提出的版本不是服务端支持的最好的版本，服务端必须通过发送一个failed消息，强制客户端转换到一个更好的版本。

当一个客户端第一次连接到一个服务端的时候，通常它会设置version为自己最偏好的版本。如果愿意，客户端可以记住和服务端沟通之后的最终版本，用于以后的连接。客户端可以依赖服务端发送一条failed消息，如果客户端或是服务端可以升级到一个更好的版本。

## Heartbeats

### Messages:

 * `ping`
   - `id`: optional string (用来和响应关联的标示符)
 * `pong`
   - `id`: optional string (和ping消息里的相同)

### Procedure:

连接建立之后，任何一方都可以发送一条ping消息。发送方可以选择在ping消息里包含一个id字段。当另一方收到ping时，它必须立即返回一条pong消息。如果收到的ping消息里包含id字段，那么pong消息必须包含相同的id字段

## Managing Data:

### Messages:

 * `sub` (客户端 -> 服务端):
   - `id`: string (任意的标示符,由客户端决定)
   - `name`: string (订阅的名字)
   - `params`: optional array of EJSON items (订阅用到的参数)
 * `unsub` (客户端 -> 服务端):
   - `id`: string (传给 'sub'的id)
 * `nosub` (服务端 -> 客户端):
   - `id`: string (传给 'sub'的id)
   - `error`: optional Error (订阅完成时抛出的错误，或是sub-not-found)
 * `added` (服务端 -> 客户端):
   - `collection`: string (collection name)
   - `id`: string (document ID)
   - `fields`: optional object with EJSON values
 * `changed` (服务端 -> 客户端):
   - `collection`: string (collection name)
   - `id`: string (document ID)
   - `fields`: optional object with EJSON values
   - `cleared`: optional array of strings (field names to delete)
 * `removed` (服务端 -> 客户端):
   - `collection`: string (collection name)
   - `id`: string (document ID)
 * `ready` (服务端 -> 客户端):
   - `subs`: array of strings (传给sub的ids,那些已经发送了初始数据的)
 * `addedBefore` (服务端 -> 客户端):
   - `collection`: string (collection name)
   - `id`: string (document ID)
   - `fields`: optional object with EJSON values
   - `before`: string or null (the document ID to add the document before,
     or null to add at the end)
 * `movedBefore` (服务端 -> 客户端):
   - `collection`: string
   - `id`: string (the document ID)
   - `before`: string or null (the document ID to move the document before, or
     null to move to the end)

### Procedure:
   
 * 客户端通过发送sub消息到服务端来指定自己关心的数据集合。

 * 任何时候，服务端收到sub消息通知，就可以发送数据消息到客户端。数据消息包括added,changed,removed消息。These messages model a local set of data the 客户端
   should keep track of.

   - 一条added消息表明一个document被添加到本地集合。document的ID通过id字段指定，其它字段通过fields字段指定。Minimongo用一种特殊的方式解析id字段，转换为Mongo document的_id字段

   - 一条changed消息表明本地集合的一个document发生变化，可能是一些字段有了新值，或是一些字段被移除。id字段是发生变化document的ID。如果出现fields对象，表明有新值的字段。cleared字段包含被移除字段的数组。

   - 一条remove消息表明本地集合的一个document被删除，id字段是document的ID。
     

 * collection可能是排序的，也可能没有。如果排序了，added消息会被addedBefore替换，before字段会包含一个document的ID。如果一个document被添加到最后，before字段设为null。对于一个给定的collection，服务端只发送added消息或是addedBefore消息，不能混用。对使用addedBefore消息的collection只使用movedBefore消息。
   
   注意：Meteor当前并没有使用排序的DDP消息。以后可能会使用。

 * 客户端保存着每一个collection的一组数据，每个订阅并不是独立的数据存储，而是重叠订阅，这样服务端会发送一个collection数据合并之后的数据。例如：如果订阅A说document x包含字段`{foo: 1, bar: 2}`，而订阅B说document x 包含字段`{foo: 1, baz:3}`，那么客户端会接到通知x 包含字段`{foo: 1, bar: 2, baz: 3}`。如果不同订阅的字段值存在冲突，服务端应该发送一个可能值。
   

 * 当一个或多个订阅发送初始数据完之后，服务端会发送一条ready消息，带着它们的ID。

## Remote Procedure Calls:

### Messages:

 * `method` (客户端 -> 服务端):
   - `method`: string (方法名)
   - `params`: optional array of EJSON items (方法的参数)
   - `id`: string (任意的标示符，由客户端决定，标识这次方法调用)
   - `randomSeed`: optional JSON value (一个任意的seed，由客户端决定，用于生成伪随机数)
 * `result` (服务端 -> 客户端):
   - `id`: string (传给'method'的id)
   - `error`: optional Error (方法抛出的错误，或是method-not-found)
   - `result`: optional EJSON item (方法的返回值，如果存在的话)
 * `updated` (服务端 -> 客户端):
   - `methods`: array of strings (传给'method'的ids, 那些已经反映到data消息的方法调用的ID)

### Procedure:

 * 客户端发送一条method消息到服务端
   
 * 服务端返回一条result消息给客户端，附带方法调用的结果或是对应的错误。

 * 方法调用可以影响到客户端已经订阅的数据。当服务端把基于这次方法调用的相关数据消息都发送给客户端之后，服务端应该发送一条updated消息给客户端，附带method的ID

 * 对一次方法调用，result和updated消息的顺序并没有特别规定。

 * 客户端可以提供一个randomSeed JSON值。如果提供了，这个值被用来生成伪随机数。通过使用相同的seed和相同的算法，客户端和服务端可以生成相同的伪随机数。特别是，用来生成新创建document的id。如果没有提供randomSeed,那么服务端和客户端生成的值将会不一样。

 * 当前版本，期望randomSeed是一个字符串，对应的生成算法还没有文档。将来可能会正式规范。

## Errors:

出现在result和nosub消息里可选的error字段是一个对象，包含下面的字段：

 * `error`: string (之前是number，参见附录3)
 * `reason`: optional string
 * `details`: optional string


这种 Error 用来代表method或subscription抛出的错误，也可能是试图订阅未知的subscription或是调用未知的method。

其它从客户端发送到服务端的错误消息，会导致客户端收到一条error消息 msg: 'error'，包括：

 * 发送的消息不是合法的JSON对象
 * 未知的msg类型
 * 其它错误形式的客户单请求(没有包含必填字段)
 * 第一条消息发送的不是connect消息，或是发送connect消息作为未初始化消息

error 消息包含下面的字段：

 * `reason`: 描述错误的字符串
 * `offendingMessage`: if the original message parsed properly, it is included
   here




## 附录：EJSON


EJSON 比JSON内置的JSON数据类型要多。增加了下面：

**Dates:**

    {"$date": MILLISECONDS_SINCE_EPOCH}

**Binary data:**

    {"$binary": BASE_64_STRING}

(The base 64 string has `+` and `/` as characters 62 and 63, and has no maximum line length)

**Escaped things** that might otherwise look like EJSON types:

    {"$escape": THING}

For example, here is the JSON value `{$date: 10000}` stored in EJSON:

    {"$escape": {"$date": 10000}}

Note that escaping only causes keys to be literal for one level down; you can
have further EJSON inside.  For example, the following is the key `$date` mapped
to a Date object:

    {"$escape": {"$date": {"$date": 32491}}}

**User-specified types:**

    {"$type": TYPENAME, "$value": VALUE}

Implementations of EJSON should try to preserve key order where they can.  Users
of EJSON should not rely on key order, if possible.

> MongoDB relies on key order.  When using EJSON with MongoDB, the
> implementation of EJSON must preserve key order.


## Appendix 2: randomSeed backward/forward compatibility

DDP pre2 增加了randomSeed，但是并没有破坏前后的兼容性。

如果客户端和服务端产生的document不一致，Meteor会解决冲突。这可能会导致UI闪烁，客户端的值发生变化，来反映服务端的变化，但是最终结果是正确的。

Consistent id generation / randomSeed does not alter the syncing process, and
thus will (at worst) be the same:


* 如果客户端和服务端都不支持randomSeed, we will get the
classical/flicker behavior.

* 如果客户端支持randomSeed而服务端不支持，那么服务端会忽略randomSeed，就像它会忽略DDP 方法调用中未知的属性。会生成不一样的id,但会通过同步修正。

* 如果服务端支持randomSeed而客户端不支持，那么服务端会生成unseeded random values(提供randomSeed是可选的)；会生成不一样的id,同样会通过同步修正。

* 如果客户端和服务端都支持randomSeed，但是生成了不一样的ID，不论是由于生成过程的bug还是stub 的行为与服务端不一致，同步会进行修正。

* 如果客户端和服务端都支持randomSeed，通常生成的ID是一样的，and syncing will be a no-op


## Appendix 3: Error message `error` field can be a string or a number

In the `pre1` and `pre2` specifications for DDP, the `error` field on error
messages was documented as a number, and in practice often matched the closest
HTTP error code.

It is typically better to be specific about the type of error being returned, so
new code should use strings like `'wrong-password'` instead of `401`. To support
both the new string error codes and the old number codes, a DDP 客户端 should
accept a string or a number in this field. Many meteor packages, including some
core packages, still use the old numeric codes.


## Version History

`pre1` DDP的第一版本

`pre2` 增加了 keep-alive (ping & pong 消息), 和 randomSeed.

`1` 可以看做是第一个官方版DDP。The type of the
error code field has been clarified to reflect the behavior of existing 客户端s
and 服务端s.

参考链接：

- [https://atmospherejs.com/meteor/ddp](https://atmospherejs.com/meteor/ddp)









