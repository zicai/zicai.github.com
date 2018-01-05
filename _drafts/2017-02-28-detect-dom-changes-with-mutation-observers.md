---
layout: post
category : javascript
title: "Mutation Observers：响应 DOM 的变化"
tagline: "Supporting tagline"
tags : [DOM]
---

### 问题
动态网页中很常见的一个需求：监测 DOM 的变化，并根据变化做出响应。你要怎么做？

- setTimeout ?

## Mutation Events
DOM3 event 规范中定义了 Mutation events。它给网页提供了一种在 DOM 发生变化(例如：`DOMNodeRemoved`，`DOMAttrModified` 等)时收到通知的机制。

一个简单的示例：

```javascript
document.addEventListener('DOMNodeInserted', function(e){
	// handle changes one-by-one. No idea how many more are coming
})
```

但是 Mutation events 存在一些缺点：

- Slow
- Crashy
- Fire too often

所以被标记为 deprecated。DOM4 event 规范提出了 Mutation Observer 用来替代 Mutation events。


## Mutation Observer
使用 Mutation Observer，开发者可以响应 DOM 的变化。

与 Mutation Events 最大的不同在于，Mutation Observer 更高效。Mutation Events 以同步的方式，每次变化都会触发对应的事件。而使用 Mutation Observer 监测某个节点的变化，直到 DOM 变化停止时，才会执行一次回调，同时把 DOM 所有变化的列表传给回调函数，开发者可以循环列表，选择想要响应的变化。


同时又不影响浏览器性能。

先来看一个简单的示例：

```javascript
// select the target node
var target = document.querySelector('#some-id');
 
// create an observer instance
var observer = new MutationObserver(function(mutations) {
    mutations.forEach(function(mutation) {
        console.log(mutation.type);
    });
});
 
// configuration of the observer:
var config = { attributes: true, childList: true, characterData: true }
 
// pass in the target node, as well as the observer options
observer.observe(target, config);
 
// later, you can stop observing
observer.disconnect();
```

### Mutation Observer 构造函数

`MutationObserver()` 构造函数用来创建一个新的 DOM mutation observer。

```javascript
var observer = new MutationObserver(function callback)
```

每次 DOM 变化之后，都会调用回调函数。回调函数接收两个参数：

- 一个对象数组，每个对象都是 `MutationRecord` 类型
- 当前 `MutationObserver` 实例


### `MutationObserver` 实例方法：

- `void observe(Node target, MutationObserverInit options)`
- `void disconnect()`
- `Array takeRecords()`

其中 `observe()` 方法用来将 `MutationObserver` 实例注册到某个节点上。需要传入两个参数：要监测变化的 DOM 节点和一个 `MutationObserverInit` 对象。 `MutationObserverInit` 用来配置监测哪些类型的 DOM 变化。它可以配置的属性包括：

- childList：是否监测目标元素的子元素的增删
- attributes：是否监测目标元素的属性
- characterData：是否监测目标元素的节点内容
- subtree：是否监测所有后代节点
- attributeOldValue：是否记录属性变化前的值
- characterDataOldValue：是否记录节点内容变化前的值
- attributeFilter：要监测的属性名数组

`disconnect()` 用来停止 `MutationObserver` 实例接收 DOM 变化的通知。

`takeRecords()` 清空 `MutationObserver` 实例的 record 队列，并返回里面的内容。

### MutationRecord 对象

DOM 每次变化都会生成一条变动记录，对应一个 MutationRecord 对象，对象包含变动相关的信息，包括：

- type：'attributes'、'characterData'、'childList'
- target：根据 type 不同而不同
- addedNodes
- removedNodes
- previousSibling
- nextSibling
- attibuteName
- oldValue


参考资料：

- https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver
- https://developer.mozilla.org/en-US/docs/Web/API/MutationRecord
- https://developers.google.com/web/updates/2012/02/Detect-DOM-changes-with-Mutation-Observers
- https://hacks.mozilla.org/2012/05/dom-mutationobserver-reacting-to-dom-changes-without-killing-browser-performance/
- [Mutation Summary](https://github.com/rafaelw/mutation-summary) is a JavaScript library that makes observing changes to the DOM fast, easy and safe. 它也是基于 Mutation Observer。
- [DOM Mutation Observers & The Mutation Summary Library](https://www.youtube.com/watch?v=eRZ4pO0gVWw) 












