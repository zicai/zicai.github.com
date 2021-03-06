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


## 权限声明

host permission 和模式匹配 https://developer.chrome.com/extensions/match_patterns

## 插件结构
https://developer.chrome.com/extensions/overview

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

To find pages in the extension, use chrome.extension methods such as getViews() and getBackgroundPage(). Once a page has a reference to other pages within the extension, the first page can invoke functions on the other pages, and it can manipulate their DOMs.

#### pop 页面

生命周期

变量共享？
函数共享？

#### option 页面
https://developer.chrome.com/extensions/optionsV2

让用户可以自定义插件的一些行为。权限声明如下：

```json
{
  "name": "My extension",
  ...
  "options_ui": {
    // Required.
    "page": "options.html",
    // Recommended.
    "chrome_style": true
  },
  ...
}
```


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

内容脚本的执行环境：
内容脚本在一个特殊环境中执行，称之为 isolated world。它可以访问被注入页面的 DOM，但是不能访问页面创建的任何 JavaScript 变量和函数。看起来就像页面中没有其他 JS 执行一样。反过来也一样，页面中的 JavaScript 也不能访问内容脚本定义的任何变量和函数。这样就避免了冲突。

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

如果想在内容脚本中使用扩展包内部的资源，需要设置 [web_accessible_resources](https://developer.chrome.com/extensions/manifest/web_accessible_resources) 

## API 列表

[https://developer.chrome.com/extensions/api_index](https://developer.chrome.com/extensions/api_index) 展示了完整的 API 列表：

### Stable APIS

- accessibilityFeatures
- alarms：调度代码周期性的运行
- bookmarks：管理、操纵书签
- browserAction：地址栏右侧的 icon，可以包括：tooltip、badge 和 popup
- browsingData：用来清除用户浏览数据
- certificateProvider
- commands：添加键盘快捷键
- contentSettings：在每个站点自定义 chrome 的行为
- contextMenus：右键快捷菜单
- cookies
- debugger
- declarativeContent
- desktopCapture
- devtools.inspectedWindow：与审查窗口交互
- devtools.network
- devtools.panels
- documentScan
- downloads
- enterprise.deviceAttributes
- enterprise.platformKeys
- events
- extension
- extensionTypes
- fileBrowserHandler
- fileSystemProvider
- fontSettings
- gcm
- history
- i18n
- identity：用来获取 OAuth2 token
- idle
- input.ime
- instanceID
- management
- networking.config
- notifications
- omnibox
- pageAction
- pageCapture
- permissions
- platformKeys
- power
- printerProvider
- privacy	
- proxy
- runtime
- sessions
- storage
- system.cpu
- system.memory
- system.storage
- tabCapture
- tabs：和浏览器的标签系统进行交互。
- topSites
- tts
- ttsEngine
- types
- vpnProvider
- wallpaper
- webNavigation
- webRequest：观测和分析网络流量，拦截或修改进行中的请求。
- webstore
- windows

### Beta APIS

- declarativeWebRequest	

### Dev APIS

- automation	
- processes
- signedInDevices	

### Experimental APIS

### API 约定
除非文档说明，否则 chrome.* API 都是异步的

## 常用 API 介绍

### 消息传递
[draft]()

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

```javascript
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

### webRequest
[https://developer.chrome.com/extensions/webRequest](https://developer.chrome.com/extensions/webRequest)

webRequest API 可以用来观察和分析网络流量，还可以拦截和修改进行中的的请求。

需要权限：'webRequest' 和 host 权限。如果想要拦截请求，还需要 'webRequestBlocking' 权限。例如

```
"permissions": [
          "webRequest",
          "*://*.google.com/"
        ]
```

可用 scheme：http://, https://, ftp://, file://, ws:// , wss:// ,chrome-extension://

请求的生命周期

- onBeforeRequest
- onBeforeSendHeaders：扩展可以通过该事件添加、修改、删除请求头。
- onSendHeaders：在请求头最终发送到网络之前触发该事件
- onHeadersReceived：收到一个响应头时触发。由于跳转和 authentication request，所以一个请求可能多次触发该事件。扩展可以通过该事件添加、修改和删除响应头。
- onAuthRequired
- onBeforeRedirect
- onResponseStarted：接收到响应体的第一个字节时触发
- onCompleted
- onErrorOccurred

web request API 确保对于每个请求最后一个触发的事件不是 onCompleted 就是 onErrorOccurred。只有一个例外，如果某个请求被重定向到 data:// URL，最后一个事件是 onBeforeRedirect。

webRequest API 根据请求的生命周期定义了一系列的事件。开发者通过给这些事件注册监听器来监测和分析流量。

注册事件监听器
要想给一个请求注册事件监听器，需要使用普通 `addListener()` 函数的变种。除了要指定回调函数之外，还必须指定一个过滤参数，和一个可选的额外信息参数。

例如：给 onBeforeRequest 事件注册监听函数：

```javascript
chrome.webRequest.onBeforeRequest.addListener(callback, filter, opt_extraInfoSpec);
```

上面三个参数的格式分别如下：

```javascript
var callback = function(details) {...};
var filter = {...};
var opt_extraInfoSpec = [...];
```
callback 接收一个对象作为参数。对象包含当前请求的信息，信息与当前事件类型和 opt_extraInfoSpec 有关。

如果 opt_extraInfoSpec 数组包含字符串 'blocking'(只有特定事件允许)，callback 以同步方式进行处理（前提是有 webRequestBlocking 权限）。也就是说，请求会被阻塞直到 callback 返回。在这种情况下，回调可以返回一个 `webRequest.BlockingResponse` 来决定请求后面的生命周期。

例如：返回 `{cancel:true}` 可以取消请求，返回 `{redirectUrl:'someurl'}` 可以重定向请求。

```javascript
chrome.webRequest.onBeforeRequest.addListener(
        function(details) { return {cancel: true}; },
        {urls: ["*://www.evil.com/*"]},
        ["blocking"]);
```

根据不同的事件类型，你可以在 opt_extraInfoSpec 数组中提供不同的字符串，来获取有关请求更多的信息。例如：要在 onSendHeaders 事件中获取请求头：

```javascript
chrome.webRequest.onSendHeaders.addListener(callback, {urls: ['<all_urls>']}, ['requestHeaders']);
```

其中 filter 用来从多角度过滤触发事件的请求：

- URLs
- Types
- Tab ID
- Window ID

注意：

- 该 API 目前没办法获取响应 body
https://bugs.chromium.org/p/chromium/issues/detail?id=487422
- 通过 onBeforeSendHeaders 修改请求头时，有一些请求头不允许修改。
- 通过 onHeadersReceived 修改响应头后，在 DevTools 中是看不到的。[https://medium.com/@requestly_ext/list-of-headers-not-allowed-to-be-modified-by-chrome-extensions-c850b5c02fff](https://medium.com/@requestly_ext/list-of-headers-not-allowed-to-be-modified-by-chrome-extensions-c850b5c02fff)

## 常见需求

### 获取当前 tab id


参考资料：

- https://developer.chrome.com/extensions/overview
- 《Chrome 扩展及应用开发》http://www.ituring.com.cn/book/1421
- https://crxdoc-zh.appspot.com/extensions/
