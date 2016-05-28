---
layout: post
category : lessons
title: "node.js express 学习笔记"
tagline: "Supporting tagline"
tags : [node.js, express]
---


## express 是什么
Fast, unopinionated, minimalist web framework for Node.js

## 安装

```
npm install express --save
```

### hello world 示例

```
var express = require('express');
var app = express();

app.get('/', function (req, res) {
  res.send('Hello World!');
});

app.listen(3000, function () {
  console.log('Example app listening on port 3000!');
});
```

### Express application generator

使用 `express-generator` 可以快速生成项目骨架。

```
npm install express-generator -g
express -h
express myapp // 创建 myapp 目录，并在其中生成项目骨架
```

`express myapp` 默认使用 `jade` 模板引擎、plain css。可以改为使用 `handlebars` 模板引擎和 Sass：

```
express myapp --hbs --css sass
```

安装依赖并运行：

```
cd myapp && npm install
DEBUG=myapp:* npm start
```

## 调试
Express 内部使用 debug 模块来记录信息，例如：路由匹配、调用的中间件函数、application mode、请求响应周期。

要查看 express 所有内部日志，启动 APP 时，将环境变量 DEBUG 设置为 express:* 。如下：

```
DEBUG=express:* node app.js
```

还可以设置为

```
DEBUG=express:router
DEBUG=express:application
```
更多信息参见：[debug](https://www.npmjs.com/package/debug)

## 路由

```
app.METHOD(PATH, HANDLER)
```

其中 `app` 是 express 实例。

express 使用 path-to-regexp https://www.npmjs.com/package/path-to-regexp 匹配路径。

query string 不属于 路由路径

express.Router() 模块化路由

## 请求

### 文件上传
从 express 4 开始，默认在 req 对象上是访问不到 req.files 的。要想访问 req.files 对象，需要使用 multipart-handling 中间件，例如：

- [busboy](https://github.com/mscdex/busboy): A streaming parser for HTML form data for node.js
- [multer](https://github.com/expressjs/multer): Multer is a node.js middleware for handling multipart/form-data, which is primarily used for uploading files. It is written on top of busboy for maximum efficiency.
- [formidable](https://github.com/felixge/node-formidable): A node.js module for parsing form data, especially file uploads.
- [multiparty](https://github.com/andrewrk/node-multiparty): A node.js module for parsing multipart-form data requests which supports streams2
- [connect-multiparty](https://github.com/andrewrk/connect-multiparty):connect middleware for multiparty 

formidable 和 busboy 的区别？？

## 响应

### 属性

- res.app
- res.headerSent
- res.locals

### 方法

- 响应头

	```
	res.append('Set-Cookie', 'foo=bar; Path=/; HttpOnly');
	```
	注意：在 res.append() 之后调用 res.set()，会重置前面设置的响应头。
	
	```
	res.set(field [, value])
	// Aliased as res.header(field [, value]).
	```
	
	获取响应头：
	
	```
	res.get(field)
	```
	
	设置 Link 响应头：
	
	```
	res.links(links)
	```
	
	设置 Location 响应头：
	
	```
	res.location(path)
	```
	
	设置 Content-Type 响应头：
	
	```
	res.type(type)
	```
	
	设置 vary 响应头：
	
	```
	res.vary('field')
	```

- 状态码
	
	```
	res.sendStatus(statusCode)
	res.status(code)
	```

- 响应附件

	响应附件，同时设置对应的响应头
	
	```
	res.attachment();
	// Content-Disposition: attachment
	
	res.attachment('path/to/logo.png');
	// Content-Disposition: attachment; filename="logo.png"
	// Content-Type: image/png
	```
	
	
	```
	res.download(path [, filename] [, fn])
	```
	它使用 `res.sendFile()` 来传输文件
	
	```
	res.sendFile(path [, options] [, fn])
	```

- 设置 cookie

	```
	res.cookie(name, value [, options])
	```
	
	res.cookie 做的也是设置响应头
	
	删除指定 cookie
	
	```
	res.clearCookie(name [, options])
	```

- 响应内容

	```
	res.send([body]) // body 可以是 buffer 对象、字符串、对象或是数组
	res.json([body])
	res.jsonp([body])
	
	res.render(view [, locals] [, callback])
	```

- 结束响应

	```
	res.end([data] [, encoding])
	```

- 跳转

	```
	res.redirect([status,] path)
	```

- 根据请求的 Accept 请求头，响应不同内容
	
	```
	res.format()
	```
	

## 中间件
中间件可以

- 执行任何代码
- 修改请求和响应
- 结束请求-响应
- 调用下一个中间件

中间件分为下面几种：

- application-level 中间件 ：使用 app.use() 和 app.METHOD()
- Router-level 中间件 ：和上一种一样，只是它绑定到了 express.Router() 实例上
- 错误处理中间件： 总是有四个参数
- 内置中间件
- 第三方中间件

express 4.x 不再依赖 connect https://github.com/senchalabs/connect?_ga=1.90901146.1447663298.1443172894 除了express.static

express 唯一内置的中间件是 express.static。它基于 serve-static
### 内置的中间件

- express.static：用于[提供静态文件](http://expressjs.com/en/starter/static-files.html)

### 第三方中间件
- [body-parser](https://github.com/expressjs/body-parser#readme)
- cookie-parser
- serve-favicon
- morgan
- [node-sass-middleware](https://github.com/sass/node-sass-middleware)

### 使用中间件
### 编写中间件

## 错误处理

定义错误处理的中间件函数和定义其它中间件是一样的，除了错误处理函数有四个参数而不是三个：(err,req,res,next)。

### 默认的错误处理

## 集成
### 集成模板引擎

模板引擎使你可以在项目中使用静态模板文件。运行的时候，模板引擎用实际的值替换模板文件中的变量，将模板转换为 HTML 发送到客户端。

常见的模板引擎有：

- [Pug (formerly Jade)](http://jade-lang.com/)
- [Mustache](https://www.npmjs.com/package/mustache)
- [EJS](https://www.npmjs.com/package/ejs)

在 Express 中可以使用的[模板引擎列表](https://github.com/expressjs/express/wiki?_ga=1.1511859.323207346.1463184090#template-engines)

模板引擎对比：[](https://strongloop.com/strongblog/compare-javascript-templates-jade-mustache-dust/)

为了渲染模板文件，需要设置下面两个应用属性

- `view` : 模板文件的目录。例如：`app.set('view','./views')`。默认的 `view` 目录是项目根目录。
- `view engine` : 要使用的模板引擎。例如：`app.set('view engine', 'pug')`。前提是安装了引擎对应的 npm 包。Express 会自动加载对应模块，无需手动引入。

参考链接：

- http://expressjs.com/en/guide/using-template-engines.html


### 集成数据库
http://expressjs.com/en/guide/database-integration.html


## 最佳实践
### 安全
### 性能

参考链接：

- [https://github.com/expressjs/express/wiki](https://github.com/expressjs/express/wiki)