---
layout: post
category : node.js
title: "Node.js 网络编程"
tagline: "Supporting tagline"
tags : []
---


Node 提供的模块

- net：用于处理 TCP
- dgram：用于处理 UDP
- http
- https

## 构建 TCP 服务
TCP 是面向连接的协议。服务端和客户端分别提供一个套接字，两个套接字共同形成一个连接，形成会话，之后才能发送数据。

一个简单的 TCP 服务端的例子

```
const net = require('net');
const server = net.createServer();
server.on('connection', function (socket) {
    console.log('on connection');
});
server.listen(9000);
```



使用 net 模块

```
const net = require('net');
```

创建一个新的 server

```
const server = net.createServer();
```

返回的是一个 net.Server，它是一个 EventEmitter 实例，它的自定义事件有以下几种：

- close
- connection
- error
- listening

可用的方法有：

- server.address()
- server.close()：让服务器停止接受新的连接，保持已存在连接。当所有连接都结束时，服务器最终关闭，同时发射一个 close 事件。
- server.getConnections()：异步的获取当前连接数
- server.listen()：有多种格式
- server.ref()
- server.unref()

可用的属性有：

- listening：布尔值，显示服务器是否正在监听
- maxConnections


当有新的连接进入服务器时，Node.js 会创建一个 net.Socket 实例，传递给服务器的 connection 事件处理器。

net.Socket 也是 EventEmitter 实例，它的自定义事件包括：

- connect
- data：收到数据时触发，回调函数的参数 data 是 Buffer 或 String
- drain
- end
- error
- lookup
- timeout

可用的方法有：

- socket.address()
- socket.connect()
- socket.destroy()
- socket.end()
- socket.pause()：暂停读取数据
- socket.ref()
- socket.resume()：暂停之后恢复读取数据
- socket.setEncoding()
- socket.setKeepAlive()
- socket.setNoDelay()
- socket.setTimeout()
- socket.unref()
- socket.write()


可用的属性有：

- socket.bufferSize
- socket.bytesRead：已接受的字节数
- socket.bytesWritten：已发送的字节数
- socket.connecting
- socket.destroyed
- socket.localAddress
- socket.localPort
- socket.remoteAddress
- socket.remoteFamily
- socket.remotePort


### 构建 TCP 服务端





参考资料：

- 《深入浅出 Node.js》第七章
- https://nodejs.org/api/net.html






