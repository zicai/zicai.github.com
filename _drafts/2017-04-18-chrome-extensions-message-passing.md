---
layout: post
category : chrome
title: "Chrome Extensions 消息传递"
tagline: "Supporting tagline"
tags : [extension]
---


[https://developer.chrome.com/extensions/messaging](https://developer.chrome.com/extensions/messaging)

由于内容脚本运行在网页的上下文中，而不是扩展本身的上下文。所以它经常需要与扩展的其余部分进行消息的传递。 

消息传递的双方都可以监听对方的消息，然后通过相同信道回复。消息可以包含任意合法 JSON 对象。

消息传递主要有以下几种形式：

- 简单的一次请求
- 长连接
- 跨扩展通信
- 从网页发消息
- native 消息



## 一次请求

- 从内容脚本发消息到扩展使用 `chrome.runtime.sendMessage`
- 从扩展发消息到内容脚本使用 `chrome.tabs.sendMessage`，因为需要指定发送到哪个 tab

示例：

```js
// 从内容脚本发消息到扩展
chrome.runtime.sendMessage({greeting: "hello"}, function(response) {
  console.log(response);
});

// 从扩展发消息到内容脚本
chrome.tabs.query({active: true, currentWindow: true}, function(tabs) {
  chrome.tabs.sendMessage(tabs[0].id, {greeting: "hello"}, function(response) {
    console.log(response);
  });
});
```

接受消息的一方通过监听 runtime.onMessage 来处理消息。

```js
chrome.runtime.onMessage.addListener(function(request, sender, sendResponse) {
    console.log(sender.tab ?
                "from a content script:" + sender.tab.url :
                "from the extension");
    if (request.greeting == "hello")
      sendResponse({farewell: "goodbye"});
  });
```
在上面的例子中，sendResponse 是同步调用的，如果想异步调用，需要在 onMessage 事件处理器最后 `return true;`

注意：如果多个页面都在监听 onMessage 事件，只有第一个调用 sendResponse() for a particular event 的会成功发出响应，其它响应都会被忽略。

## 长连接

在内容脚本和扩展页面之间打开长连接。对应的要使用 runtime.connect 或是 tabs.connect。信道可以有个名字，用来区分不同连接。

建立连接后，双方都会得到一个 runtime.Port 对象，它用来发送和接收消息。

例如：在内容脚本中打开一个信道，发送并监听消息

```js
var port = chrome.runtime.connect({name: "knockknock"});
port.postMessage({joke: "Knock knock"});
port.onMessage.addListener(function(msg) {
  if (msg.question == "Who's there?")
    port.postMessage({answer: "Madame"});
  else if (msg.question == "Madame who?")
    port.postMessage({answer: "Madame... Bovary"});
});
```
从扩展发送消息到内容脚本也类似，只不过需要指定想要连接的 Tab，只需将上面的例子改为 tabs.connect。

为了处理 incoming 连接，需要设置 runtime.onConnect 事件处理器。这对内容脚本和扩展都一样。当一方调用 connect() 时，另一方触发 onConnect，并利用 runtime.Port 对象来发送和接受消息。例如：

```js
chrome.runtime.onConnect.addListener(function(port) {
  console.assert(port.name == "knockknock");
  port.onMessage.addListener(function(msg) {
    if (msg.joke == "Knock knock")
      port.postMessage({question: "Who's there?"});
    else if (msg.answer == "Madame")
      port.postMessage({question: "Madame who?"});
    else if (msg.answer == "Madame... Bovary")
      port.postMessage({question: "I don't get it."});
  });
});
```

### 端口的生命周期

## 跨扩展通信

## 从网页发送消息

需要在 manifest 中指定 externally_connectable，例如：

```json
"externally_connectable": {
  "matches": ["*://*.example.com/*"]
}
```

注意：URL 模式必须包含 second-level domain，不能使用 `<all_urls>`

## 安全考虑

## 有关 devtools 的消息传递
https://developer.chrome.com/extensions/devtools#solutions

devtools page 不能直接与内容脚本通信，而是要经过背景页中转。



