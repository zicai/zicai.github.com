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

## 请求

## 响应

## 错误处理

## 中间件
### 内置的中间件

- express.static：用于[提供静态文件](http://expressjs.com/en/starter/static-files.html)

### 第三方中间件
- body-parser
- cookie-parser
- serve-favicon
- morgan
- [node-sass-middleware](https://github.com/sass/node-sass-middleware)

### 使用中间件
### 编写中间件

## 集成
### 集成模板引擎
http://expressjs.com/en/guide/using-template-engines.html

- hbs

### 集成数据库
http://expressjs.com/en/guide/database-integration.html


## 最佳实践
### 安全
### 性能