---
layout: post
category : node.js
title: "Node.js 网络编程之 TCP 与 UDP"
tagline: "Supporting tagline"
tags : []
---


## 构建 TCP 服务
TCP 是面向连接的协议。服务端和客户端分别提供一个套接字，两个套接字共同形成一个连接，形成会话之后才能发送数据。

一个简单的 TCP 服务端的例子

```
const net = require('net');
const server = net.createServer();
server.on('connection', function (socket) {
    console.log('on connection');
});
server.listen(9000);
```

### net 模块

使用 net 模块

```
const net = require('net');
```

net 模块可用方法：

- `net.connect(options[, connectListener])`：工厂函数，返回一个新的 net.socket，并且根据 option 自动连接。
- `net.connect(path[, connectListener])`
- `net.connect(port[, host][, connectListener])`
- `net.createConnection(options[, connectListener])`
- `net.createConnection(path[, connectListener])`
- `net.createConnection(port[, host][, connectListener])`
- `net.createServer([options][, connectionListener])`
- `net.isIP(input)`
- `net.isIPv4(input)`
- `net.isIPv6(input)`

### net.Server 类

创建一个新的 server

```
const server = net.createServer();
```

返回的 server 是一个 net.Server 实例，同时也是一个 EventEmitter 实例，它的自定义事件有以下几种：

- `'close'`
- `'connection'`
- `'error'`
- `'listening'`

可用的方法有：

- `server.address()`
- `server.close()`：让服务器停止接受新的连接，保持已存在连接。当所有连接都结束时，服务器最终关闭，同时发射一个 `'close'` 事件。
- `server.getConnections()`：异步的获取当前连接数
- `server.listen(handle[, backlog][, callback])`
- `server.listen(options[, callback])`
- `server.listen(path[, backlog][, callback])`
- `server.listen([port][, hostname][, backlog][, callback])`
- `server.ref()`
- `server.unref()`

可用的属性有：

- `listening`：布尔值，显示服务器是否正在监听
- `maxConnections`

### net.Socket 类

net.Socket 类是对 TCP 或 local socket 的抽象。net.Socket 实例实现了 duplex stream 接口。可以由用户创建，作为客户端（与 connect()）使用，或者当有新的连接进入服务器时，Node.js 会创建一个 net.Socket 实例，传递给服务器的 `'connection'` 事件处理器。

创建一个新的 socket 对象：

```
new net.Socket([options])

// option 默认值
{
  fd: null, // file descriptor
  allowHalfOpen: false,
  readable: false,
  writable: false
}
```

net.Socket 也是 EventEmitter 实例，它的自定义事件包括：

- `'close'`：socket 完全关闭时触发一次。参数 had_error 是一个布尔值，表示是否是由于传输错误导致的关闭。
- `'connect'`：当 socket 连接成功建立时触发。
- `'data'`：收到数据时触发，回调函数的参数 data 是 Buffer 或 String。通过 socket.setEncoding() 设置编码。
- `'drain'`：当 write buffer 变为空时触发。可以用来 throttle uploads
- `'end'`：当 socket 连接的另一端发送以 FIN 包时触发
- `'error'`：发生错误时触发。紧接着会触发 `'close'` 事件
- `'lookup'`：在解析完主机名，连接之前触发。
- `'timeout'`

可用的方法有：

- `socket.address()`
- `socket.connect()`：用给定的 socket 打开一个连接
- `socket.destroy([exception])`：出现错误时才有必要调用。
- `socket.end()`：半关闭连接，即发送一个 FIN 包，服务端仍有可能发送一些数据。
- `socket.pause()`：暂停读取数据
- `socket.ref()`
- `socket.resume()`：暂停之后恢复读取数据
- `socket.setEncoding()`
- `socket.setKeepAlive()`
- `socket.setNoDelay()`
- `socket.setTimeout(timeout[, callback])`：socket 默认是不会超时。该方法用来设置超时时间，当 socket 空闲时间超过设定时间，触发 'timeout' 事件，但是连接并不会被切断。用户必须手动 `end()` 或是 `destroy()` socket
- `socket.unref()`
- `socket.write()`


可用的属性有：

- `socket.bufferSize`
- `socket.bytesRead`：已接受的字节数
- `socket.bytesWritten`：已发送的字节数
- `socket.connecting`
- `socket.destroyed`
- `socket.localAddress`
- `socket.localPort`
- `socket.remoteAddress`
- `socket.remoteFamily`
- `socket.remotePort`

## 构建 UDP 服务

dgram 模块提供了 UDP Datagram sockets 的一个实现。

使用 dgram 模块

```
const dgram = require('dgram');
```

可用的方法：

- `dgram.createSocket(options[, callback])`：创建一个 dgram.Socket 对象
- `dgram.createSocket(type[, callback])`


### dgram.Socket 类

dgram.Socket 对象是 EventEmitter 的实例。可用事件有

- `'close'`
- `'error'`
- `'listening'`
- `'message'`

可用方法：

* socket.addMembership(multicastAddress[, multicastInterface])
* socket.address()
* socket.bind([port][, address][, callback])
* socket.bind(options[, callback])
* socket.close([callback])
* socket.dropMembership(multicastAddress[, multicastInterface])
* socket.send(msg, [offset, length,] port, address[, callback])
* socket.setBroadcast(flag)
* socket.setMulticastLoopback(flag)
* socket.setMulticastTTL(ttl)
* socket.setTTL(ttl)
* socket.ref()
* socket.unref()

参考资料：


- https://nodejs.org/api/net.html
- https://nodejs.org/api/dgram.html
- 《深入浅出 Node.js》第七章





