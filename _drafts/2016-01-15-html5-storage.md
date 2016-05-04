---
layout: post
category : lessons
title: "HTML5 存储"
tagline: "Supporting tagline"
tags : [html]
---

## 历史
对于桌面应用（或者原生应用），存储用户数据（用户配置信息或者运行时状态）的方式有很多：

- 简单地存放键值对
	- 注册表
	- INI 文件
	- XML 文件
- 复杂强大的存储
	- 数据库
	- 特定的数据文件格式

	
对 Web 应用来说，在之前的很长时间内，cookie 是唯一可以用来在本地存放用户数据的方法。使用 cookie 的不足在于：

- 经济性：每个 HTTP 请求都会带上 cookie，无形中增加了流量
- 安全性：HTTP 请求中，cookie 是明文传输	
- 大小限制：cookie 大小限制在 4KB 左右

而 Web 开发者的需求是：

- 不受限制的存储空间
- 数据保存在客户端，无需在请求间来回传递
- 数据的生命周期可以跨页面，甚至重新打开浏览器也不丢失

在 HTML5 存储出现之前，出现了一些替代技术：

- IE 的 [userData](https://msdn.microsoft.com/en-us/library/ms531424(VS.85).aspx)
- Adobe 在 Flash6 引入的 Flash cookie（Local shared Objects）
- Dojo Toolkit 框架的 dojox.storage
- [Google gears](http://gearsblog.blogspot.com/2011/03/stopping-gears.html) 项目 

HTML5 存储技术：

- [Web storage]
- [Indexed Database]
- [Web SQL Database]

其中 [Web SQL Database] 已经被废弃。

## Web storage

### 如何使用

#### 增删改查

### 安全性

## IndexedDB



参考资料：



[Web storage]:(https://www.w3.org/TR/webstorage/)
[Indexed Database]:(https://www.w3.org/TR/IndexedDB/)
[Web SQL Database]:(https://www.w3.org/TR/webdatabase/)





