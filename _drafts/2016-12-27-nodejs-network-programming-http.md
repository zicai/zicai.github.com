---
layout: post
category : node.js
title: "Node.js 网络编程之 HTTP"
tagline: "Supporting tagline"
tags : []
---

引入 http 模块

```javascript
const http = require('http');
```

http 模块公开的属性有：

- `http.METHODS`：支持的 HTTP 方法列表
- `http.STATUS_CODES`：标准 HTTP 响应状态码，和简短说明
- `http.globalAgent`：Agent 的全局实例，作为默认代理用来发送所有客户端 HTTP 请求

以及三个方法（后面会分别讲到）：

- `http.createServer()`
- `http.request()`
- `http.get()`

## 构建服务端

首先调用 `http.createServer([requestListener])` 创建一个 http.Server 的实例，requestListener 函数自动绑定到 `'request'` 事件。

调用 `server.listen([port][, hostname][, backlog][, callback])` 方法，绑定监听端口和 hostname。

需要注意的是：当提供了 hostname 参数时，需要同时给出 backlog 参数（默认是 511），否则报错。而且在本地做测试时，需要先把对应的 hostname 添加到 hosts 文件中，否则也会报错（`Error: getaddrinfo ENOTFOUND`）

### http.Server 类

继承自 net.Server 类，增加了额外的一些事件：

- `'checkContinue'`
- `'checkExpectation'`
- `'clientError'`
- `'close'`
- `'connect'`
- `'connection'`
- `'request'`：每当有请求时触发。注意：一个连接上可以有多个请求（keep-alive）。
- `'upgrade'`

方法

### http.ServerResponse 类

由 HTTP server 在内部创建，而不是用户创建。传递给 'request' 事件的监听函数，作为第二个参数。

http.ServerResponse 实现了但并没有继承 writable stream 接口。


可用事件：

- `'close'`
- `'finish'`

可用方法：

* response.addTrailers(headers)
* response.end([data][, encoding][, callback])：通知服务器响应头和响应体都已发送。每个响应都必须调用该方法。
* response.getHeader(name)
* response.removeHeader(name)
* response.setHeader(name, value)
* response.setTimeout(msecs, callback)
* response.write(chunk[, encoding][, callback])
* response.writeContinue()
* response.writeHead(statusCode[, statusMessage][, headers])

可用属性：

* response.finished
* response.headersSent
* response.sendDate
* response.statusCode
* response.statusMessage



### http.IncomingMessage 类

对于 server 的请求和客户端的响应

事件

- `'aborted'`
- `'close'`

方法

属性：

* message.headers
* message.httpVersion
* message.method
* message.rawHeaders
* message.rawTrailers
* message.setTimeout(msecs, callback)
* message.statusCode
* message.statusMessage
* message.socket
* message.trailers
* message.url

## 构建客户端

首先要通过 `http.request(options[, callback])` 构建一个 http.ClientRequest 类的实例。其中 `options` 可以是一个 url 字符串（会自动通过 `url.parse()` 解析），也可以是一个对象（具体内容见下）。可选的 `callback` 参数会作为 `'response'` 事件的监听函数（一次性的）

http.ClientRequest 实例表示一个进行中的请求。它的头部已经进入队列，不过还是可以通过 `setHeader(name, value)`, `getHeader(name)`, `removeHeader(name)` 来修改请求头。

接着可以通过 `request.write()` 发送消息体。最终调用 `request.end()` 完成请求发送。

还可以使用 Node.js 为 GET 方法提供的快捷方法 `http.get(options[, callback])`。和 `http.request()` 的区别在于，它将请求方法设为 GET，同时自动调用 `request.end()` 发送请求。

为了获取响应，需要监听实例的 `'response'` 事件。监听函数唯一的参数是 http.IncomingMessage 类的实例。当收到响应头时，触发 `'response'` 事件。

如果没有监听 `'response'` 事件，那么整个响应就被丢弃。不过，一旦你添加了 `'response'` 监听函数，那么必须要消耗响应对象中的数据，可以是通过 `response.read()` 也可以是添加 `'data'` 处理器，或是调用 `resume()` 方法。否则就会一直占用内存空间。

http.ClientRequest 实现了可写流接口，同时也是一个 EventEmitter 实例，定义的事件有：


- `'abort'`
- `'aborted'`
- `'connect'`
- `'continue'`
- `'response'`：收到响应时触发。
- `'socket'`
- `'upgrade'`

http.ClientRequest 实例可用方法有：

* `request.abort()`：将请求标记为终止。响应数据被丢弃同时 socket 被释放。
* `request.end([data][, encoding][, callback])`：完成发送请求
* `request.flushHeaders()`
* `request.setNoDelay([noDelay])`
* `request.setSocketKeepAlive([enable][, initialDelay])`
* `request.setTimeout(timeout[, callback])`
* `request.write(chunk[, encoding][, callback])`


### http.Agent 类







参考资料：

- https://nodejs.org/api/http.html
- 《深入浅出 Node.js》第七章





