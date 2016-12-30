---
layout: post
category : node.js
title: "学习 Node.js 常见错误"
tagline: "Supporting tagline"
tags : []
---

## 网络相关



### Error: connect ECONNREFUSED
当客户端尝试连接到一个服务端未监听的端口时触发。

例如：

```
const net = require('net');
var client = net.connect(8124);
```

```
Error: connect ECONNREFUSED 127.0.0.1:8124
    at Object.exports._errnoException (util.js:1007:11)
    at exports._exceptionWithHostPort (util.js:1030:20)
    at TCPConnectWrap.afterConnect [as oncomplete] (net.js:1080:14)

```

通过监听 `net.connect` 返回的 socket（也就是 client）的 `'error'` 事件来捕获异常

```
client.on('error', function (e) {
    console.log(e);
});
```



### socket hang up
有两种情况：

- 你作为客户端，发起请求，没有收到及时的响应。监听客户端的 error 事件捕获该异常。
- 你作为服务端
    - 作为proxy，在你准备响应内容时，客户端关闭了请求（刷新了页面）

