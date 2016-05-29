---
layout: post
category : lessons
title: "node.js connect 学习笔记"
tagline: "Supporting tagline"
tags : [node.js, connect]
---


https://github.com/senchalabs/connect

Connect is a middleware layer for Node.js

## 常用中间件


- [body-parser](https://github.com/expressjs/body-parser#readme)
- https://www.quora.com/What-exactly-does-body-parser-do-with-express-js-and-why-do-I-need-it
- [cookie-parser](https://github.com/expressjs/cookie-parser)
- [serve-favicon](https://github.com/expressjs/serve-favicon) favicon serving middleware
- [morgan](https://github.com/expressjs/morgan) HTTP request logger middleware for node.js
- [compression](https://github.com/expressjs/compression) Node.js compression middleware
- [method-override](https://github.com/expressjs/method-override) Override HTTP verbs.
- [node-sass-middleware](https://github.com/sass/node-sass-middleware)
- [http-proxy-middleware](https://github.com/chimurai/http-proxy-middleware)


### cookie-parser

### serve-favicon

### method-override
Lets you use HTTP verbs such as PUT or DELETE in places where the client doesn't support it.

### morgan

```
morgan(format, options)
```

预定义的格式有：

- combined
- common
- dev
- short
- tiny

options 有：

- `immediate`: Write log line on request instead of response. 
- `skip`: 用来确定是否跳过日志的函数。
- `stream`: 输出流。默认是 `process.stdout`

可以通过使用 [file-stream-rotator](https://www.npmjs.com/package/file-stream-rotator) 来每天生成一个单独的日志文件。




参考文章:

- 深入浅出Node.js（七）：Connect模块解析（之一） http://www.infoq.com/cn/articles/nodejs-connect-module#
- 深入浅出Node.js（八）：Connect模块解析（之二）静态文件中间件  http://www.infoq.com/cn/articles/nodejs-8-connect-module-part-2
- What is Node.js' Connect, Express and “middleware”? http://stackoverflow.com/a/23957864