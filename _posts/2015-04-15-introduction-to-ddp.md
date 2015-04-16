---
layout: post
category : lessons
title: "DDP简介"
tagline: "Supporting tagline"
tags : [meteor,DDP]
---

DDP 是Distributed Data Protocol的缩写。Meteor 根据DDP实现了客户端和服务端。

##DDP是什么
是一个基于JSON的协议，DDP可以在任何duplex transport上实现。Meteor当前的实现是基于WebSockets和SockJS。SockJS是一个WebSockets emulation transport，当WebSockets 不可用时使用。

##DDP用来做什么
它主要来做两件事：

1. 处理Remote Procedure Calls (RPC)
2. 管理数据

##处理RPC
使用RPC，你可以调用服务端的方法，并得到返回结果。除此之外，DDP还可以：it notifies the caller after all the write operations in the method have been reflected to all the other connected clients.

例如：客户端调用一个服务端函数`transferMoney`

![https://i.cloudup.com/2fLpc3NA3a.png](https://i.cloudup.com/2fLpc3NA3a.png)

下面是DDP消息：

```
1.{"msg":"method", "method": "transferMoney", "params": ["1000USD", "arunoda", "sacha"], id": "randomId-1"}
2.{"msg": "result", id": "randomId-1": "result": "5000USD"}
3.{"msg": "updated", "methods": ["randomId-1"]}
```

1. DDP 客户端(arunoda)调用method transferMoney ，带有三个参数：1000USD, arunoda 和 sacha.
2. 转账完成后，DDP服务端(bank)发送一条消息：arunoda的账户余额。余额在result field。如果出错了，会有一个error field。
3. 然后，DDP 服务端发送另一条叫做updated的消息，带着method id,通知我：到sacha的转账已经成功。有时候，updated 消息会在result之前返回。

##管理数据
这是DDP协议的核心部分。客户端可以利用这点来订阅一个实时的数据源，并得到通知。DDP协议有三种通知：`added`,`changed`,`removed`。DDP受到MongoDB的启发，每一个数据通知(一个JSON对象)都被赋值到一个collection。

例如：假设数据源叫做`account`，用来保存所有交易。在本例中，sacha连接到自己的account，获取自己的交易记录。arunoda转账之后，sacha会收到一个新的交易。数据流如下：

![https://i.cloudup.com/36TF0RmTLM.png](https://i.cloudup.com/36TF0RmTLM.png)

DDP消息如下：

```
1.{"msg": "sub", id: "random-id-2", "name": "account", "params": ["sacha"]}
2.{"msg": "added", "collection": "transactions", "id": "record-1", "fields": {"amount": "50USD", "from": "tom"}}
  {"msg": "added", "collection": "transactions", "id": "record-2", "fields": {"amount": "150USD", "from": "chris"}}
3.{"msg": "ready": "subs": ["random-id-2"]}
4.{"msg": "added", "collection": "transactions", "id": "record-3", "fields": {"amount": "1000USD", "from": "arunoda"}}
```

1. DDP 客户端(sacha)发送一个订阅请求到自己的账户
2. 他会已发生交易的通知
3. DDP服务端发送完所有交易之后，会发送一个叫做`ready`的特殊消息。`ready`消息指明订阅的所有初始数据都已经发送完成。
4. 过了一会儿，arunnoda转账之后，sacha会收到一条added 消息。

同样的，DDP服务端也可以发送changed 和removed通知。例如：

```
//changed
{"msg": "changed", collection": "transactions", "id": "doc_id", "fields": {"amount": "300USD"}}

//removed
{"msg": "removed", "collection": "transactions", "id": "doc_id"}
```

##理解和分析DDP
理解和分析DDP非常重要，利于你裂解Meteor的工作原理。要想看到真实的DDP消息，可以安装`ddp-analyzer`

[DDP Analyzer](http://meteorhacks.com/discover-meteor-ddp-in-realtime.html)。

![https://i.cloudup.com/IsUVXUOspa.png](https://i.cloudup.com/IsUVXUOspa.png)











