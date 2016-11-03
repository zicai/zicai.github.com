---
layout: post
category : html
title: "HTML5 本地存储"
tagline: "Supporting tagline"
tags : [HTML5,storage]
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

- 每个 HTTP 请求都会带上 cookie，无形中增加了流量
- HTTP 请求中，cookie 是明文传输	
- 大小限制：cookie 大小限制在 4KB 左右

而 Web 开发者的需求是：

- 不受限制的存储空间
- 数据保存在客户端，无需在请求间来回传递
- 数据的生命周期可以跨页面，甚至重新打开浏览器也不丢失

在 HTML5 本地存储出现之前，出现了一些替代技术：

- IE 的 [userData](https://msdn.microsoft.com/en-us/library/ms531424(VS.85).aspx)
- Adobe 在 Flash6 引入的 Flash cookie（Local shared Objects）
- Dojo Toolkit 框架的 dojox.storage
- [Google gears](http://gearsblog.blogspot.com/2011/03/stopping-gears.html) 项目 

HTML5 本地存储技术：

- [Web storage]
- [Indexed Database]
- [Web SQL Database]

其中 [Web SQL Database] 已经被废弃。

使用 HTML5 本地存储技术主要是出于两点考虑：

1. 离线情况下也可以使用
2. 提升性能

在详细的介绍每种技术之前，先来看 HTML5 本地存储方案的共同点：

## 共同特征

- 保存在客户端设备
- 受同源策略限制
- 限额：每一种存储方案在每一个源上可用的空间是有限额的。[	Quota Management API 草案](https://www.w3.org/TR/quota-api/)
- 事务：两种数据库存储方案支持事务
- 异步与同步：大多数存储方案都支持同步和异步两种模式。


接下来看每种存储技术的细节：

## Web storage

Web Storage API 是浏览器提供的，可以用来保存键值对。

Web Storage 给每个域分配了独立的存储空间，包含两种不同的机制：

- sessionStorage：通过 `window.sessionStorage` 访问。在同一个浏览器窗口中打开的相同站点的任何页面都可以访问。
- localStorage：通过 `window.localStorage` 访问。数据可以跨窗口共享，即使窗口关闭，数据也不会清空。

访问 `window.sessionStorage` 和 `window.localStorage` 都会返回 `Storage` 对象实例，通过它对数据进行增删改查。需要注意的是，二者返回的是独立的 `Storage` 对象实例。

`Storage` 对象的方法和属性：

```
Storage.length			// 返回 Storage 对象中保存的键的数量
Storage.getItem()		// 传入键名，返回键值
Storage.setItem()		// 传入键名和键值，保存到 storage，如果键名已存在，则更新键值
Storage.removeItem()	// 传入键名，从 storage 中移除对应的键值对
Storage.clear()			// 清空 storage
Storage.key()				// 传入数字 n，返回第 n 个键名
```

当 storage 中保存的数据发生变化时，就会触发 `StorageEvent`。需要注意的是：导致变化的页面，不会触发该事件。该机制可以用来在相同域下不同窗口之间同步数据变化。


Web storage 适用于存储少量的数据，当需要存取数据量较大的结构化数据时，使用 IndexedDB 是更好的方案。
## IndexedDB

客户端可以使用 IndexedDB 来保存数量较大的结构化数据。它使用索引来提升查询效率。


参考资料：

- [Client-Side Storage](http://www.html5rocks.com/en/tutorials/offline/storage/)
- ["Offline": What does it mean and why should I care?](https://www.html5rocks.com/en/tutorials/offline/whats-offline/)
- [https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)

[Web storage]:(https://www.w3.org/TR/webstorage/)
[Indexed Database]:(https://www.w3.org/TR/IndexedDB/)
[Web SQL Database]:(https://www.w3.org/TR/webdatabase/)





