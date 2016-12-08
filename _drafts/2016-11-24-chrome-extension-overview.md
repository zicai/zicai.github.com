---
layout: post
category : chrome
title: "Google Chrome 插件基础"
tagline: "Supporting tagline"
tags : []
---

### 词汇的翻译

- content scripts：内容脚本
- background page：背景页

## 文件组成

manifest

extension UI

- browser action：当扩展和大多数页面有关时
- page action：和某些页面有关时

## 插件结构

- 背景页（background page）
- 用户接口页（UI page），为了方便理解，可以叫做前景页。
- 内容脚本（content scripts）

### 背景页
负责扩展的主逻辑。
背景页可以分为两种：

- persistent background page
- event page

#### event page
https://developer.chrome.com/extensions/event_pages

需要在 manifest 中 background 中显式声明 persistent:false。

声明周期：
event page 会在需要时加载。空闲时被卸载。下面是一些会导致 event page 加载的事件：

- 首次安装或是更新到新版本
- event page 监听某个事件，事件触发时
- content script 或是其他扩展发送一个消息
- 扩展的其它页面调用 runtime.getBackgroundPage

可以通过 chrome 任务管理器观察 event page 的生命周期。加载时，扩展会出现在进程列表中。

### 前景页

- popup
- option
- override page
- 通过 tabs.create 或 window.open 打开任何页面


The HTML pages inside an extension have complete access to each other's DOMs, and they can invoke functions on each other.
Because all of an extension's pages execute in same process on the same thread, the pages can make direct function calls to each other.

#### pop 页面

生命周期

变量共享？
函数共享？

#### override page
https://developer.chrome.com/extensions/override

通过 override page，可以用插件里的 HTML 文件替换某些 chrome 默认提供的页面。包括：

- 书签管理器：chrome://bookmarks
- 历史：chrome://history
- 新标签页：chrome://newtab

注意：一个插件只能替换一个页面。

匿名窗口有些特殊，匿名窗口的新标签页不能被替换。

需要在 manifest 文件中注册要替换的页面：

```
{
  "name": "My extension",
  ...
  "chrome_url_overrides" : {
    "pageToOverride": "myPage.html"
  },
  ...
}
```

其中，要将 `pageToOverride` 替换成下面三者之一：

- bookmarks
- history
- newtab

**提示：**

- 尽可能让你的页面体积小，且加载迅速
- 给你的页面指定标题 `<title>New Tab</title>`，否则用户会看到页面的 URL
- 不要依赖页面会获得输入焦点，因为当用户打开新标签时，地址栏总是先获得焦点
- 不要试图模仿默认的新标签页


### 内容脚本

https://developer.chrome.com/extensions/content_scripts

和页面进行交互。可以把它看做网页的一部分
run in the context of a web page and not the extension

它不能：

- 使用 chrome.* API 除了：
    - extension ( getURL , inIncognitoContext , lastError , onRequest , sendRequest )
    - i18n
    - runtime ( connect , getManifest , getURL , id , onConnect , onMessage , sendMessage )
    - storage
- 使用扩展页面定义的变量和函数
- 使用宿主页面或是其它 content script 定义的变量和函数

可以：

- 和extension 通信
- 发起 Ajax 到与扩展相同的站点
- 通过共享 DOM 与宿主页面通信

## 常用 API
### 消息传递
[https://developer.chrome.com/extensions/messaging](https://developer.chrome.com/extensions/messaging)

由于内容脚本运行在网页的上下文中，而不是扩展本身的上下文。所以它经常需要与扩展的其余部分进行消息的传递。 

消息传递的双方都可以监听对方的消息，然后通过相同信道回复。消息可以包含合法 JSON 对象。

消息传递主要有以下几种形式：

- 简单的一次请求
- 长连接
- 跨扩展通信
- 从网页发消息
- native 消息



#### 一次请求

- 从内容脚本发消息到扩展使用 `chrome.runtime.sendMessage`
- 从扩展发消息到内容脚本使用 `chrome.tabs.sendMessage`，因为需要指定发送到哪个 tab

示例：

```
// 从内容脚本发消息到扩展
chrome.runtime.sendMessage({greeting: "hello"}, function(response) {
  console.log(response.farewell);
});

// 从扩展发消息到内容脚本
chrome.tabs.query({active: true, currentWindow: true}, function(tabs) {
  chrome.tabs.sendMessage(tabs[0].id, {greeting: "hello"}, function(response) {
    console.log(response.farewell);
  });
});
```



接受消息的一方 chrome.runtime.onMessage.addListenner(callback) 。可以同步返回响应也可以异步

#### 长连接

### 存储数据
通常使用以下三种方法来存储数据：

- localStorage
- Web SQL Database
- chrome.storage API

以上几种方式都不会对数据加密。

#### chrome.storage API
[https://developer.chrome.com/extensions/storage](https://developer.chrome.com/extensions/storage)

|功能|存储、检索、跟踪用户数据的变化|
|:---|:--|
|权限|"storage"|
|内容脚本|完全支持|

chrome.storage API 提供了三种存储区域（StorageArea）：

- chrome.storage.sync
	- 会根据用户当前在 chrome 上登录的 Google 账号自动同步数据
	- 最大容量：102400 字节，单个项最大：8129 字节，最多包含 512 个项
- chrome.storage.local
	- 单机存储
	- 最大容量：5242880 字节（5MB）
- chrome.storage.managed：只读

chrome.storage API 与 localStorage 的区别在于：内容脚本可以直接读取存储 API 的数据，而不必通过背景页。而如果内容脚本直接读取 localStorage ，所读取到的数据时用户当前页面所在域中的。所以通常的解决办法是内容脚本通过 runtime.sendMessage 和背景页通信，由背景页读写扩展所在域（通常是 chrome-extension://extension-id）的 localStorage ，然后再传递给内容脚本


StorageArea 包含以下方法：

- get
- getBytesInUse
- set
- remove
- clear

```
// 保存数据
chrome.storage.sync.set({'value': theValue}, function() {
	// Notify that we saved.
	message('Settings saved');
});

// 查询数据
chrome.storage.sync.get(key, function (obj) {
  // 
});

// 监听数据变化
chrome.storage.onChanged.addListener(function(changes, namespace) {
  for (key in changes) {
    var storageChange = changes[key];
    console.log('Storage key "%s" in namespace "%s" changed. ' +
      'Old value was "%s", new value is "%s".',
      key,
      namespace,
      storageChange.oldValue,
      storageChange.newValue);
  }
});
```

### 文件系统

### Cookie
[https://developer.chrome.com/extensions/cookies](https://developer.chrome.com/extensions/cookies)

|功能|查询、修改 cookie，监控 cookie 变化|
|:---|:--|
|权限|"cookies" + host permissions|

例如：

```
"permissions": [
	"cookies",
	"*://*.google.com"
]
```


- chrome.cookies.get：读取单个 cookie
- chrome.cookies.getAll：读取一个 cookie store 中的所有 cookie
- chrome.cookies.set：设置 cookie
- chrome.cookies.remove：删除 cookie

设置和删除 cookie 时，需要扩展对 URL 有访问权限

导致 cookie 变化的原因有以下几种：

- "evicted"：由于垃圾回收导致的 cookie 移除
- "expired"：由于过期导致的 cookie 移除
- "expired_overwrite"：用已过期时间重写导致的 cookie 移除
- "explicit"：新添加或是调用 chrome.cookies.remove 导致的 cookie 变化
- "overwrite" ：调用 set 导致的 cookie 变化

注意：更新 cookie 属性被实现为两步：首先移除要更新的 cookie，原因标记为 "overwrite"，然后写入新 cookie，原因标记为 "explicit"

监控 cookie 的变化 `chrome.cookies.onChanged`，例如：

```
chrome.cookies.onChanged.addListener(function(){

})

```



参考资料：

- https://developer.chrome.com/extensions/overview
- 《Chrome 扩展及应用开发》http://www.ituring.com.cn/book/1421
- https://crxdoc-zh.appspot.com/extensions/
