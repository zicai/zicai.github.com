---
layout: post
category : node.js
title: "理解 Node.js 事件"
tagline: "Supporting tagline"
tags : [event]
---


大多数 Node.js 核心 API 都是围绕异步事件驱动的架构来实现的。一些特定的对象（称为 emitters）周期性的发射命名事件，导致函数对象（称为 listeners）被调用。

例如：

- 每次有链接进来，net.Server 对象就发射事件
- 当文件打开时，fs.ReadStream 发射事件
- 当数据可以读取时，stream 发射事件

所有发射事件的对象都是 EventEmitter 的实例。这些对象暴露 `eventEmitter.on()` 函数，允许一个或多个函数附加到该对象发射的命名事件上。

当 EventEmitter 对象发射一个事件，所有注册到该事件的监听函数都会被同步调用。所有这些函数的返回值都会被忽略和抛弃。


简单的例子：

```javascript
const EventEmitter = require('events');

class MyEmitter extends EventEmitter {}

const myEmitter = new MyEmitter();
myEmitter.on('event', () => {
    console.log('an event occurred!')
});
myEmitter.emit('event');
```


## 传递参数和 this 到监听函数

`eventEmitter.emit()` 方法允许传入任意数量的参数给监听函数。

要注意的是，监听函数的写法会影响其 this 的值。

- 用普通函数时，this 指向的是监听器所附加的 EventEmitter

```javascript
const myEmitter = new MyEmitter();
myEmitter.on('event', function(a, b) {
  console.log(a, b, this);
  // Prints:
  //   a b MyEmitter {
  //     domain: null,
  //     _events: { event: [Function] },
  //     _eventsCount: 1,
  //     _maxListeners: undefined }
});
myEmitter.emit('event', 'a', 'b');
```

- 用箭头函数时，this 就不再指向 EventEmitter 实例。

```javascript
const myEmitter = new MyEmitter();
myEmitter.on('event', (a, b) => {
  console.log(a, b, this);
  // Prints: a b {}
});
myEmitter.emit('event', 'a', 'b');
```

## 同步 VS 异步

EventListener 按照监听函数的注册顺序，同步调用每个监听函数。这是为了保证合适的事件序列以及避免竞态和逻辑错误。适当的时候，可以通过 `setImmediate()` 或 `process.nextTick()` 将监听函数切换到异步模式。


```javascript
const myEmitter = new MyEmitter();
myEmitter.on('event', (a, b) => {
  setImmediate(() => {
    console.log('this happens asynchronously');
  });
});
myEmitter.emit('event', 'a', 'b');
```

## 处理单次事件

用 `eventEmitter.on()` 注册的监听函数，事件每次触发时都会执行一遍。而使用 `eventEmitter.once()` 方法可以注册最多只执行一次的监听函数，事件第一次触发时，监听函数被 unregistered 并调用。

```javascript
const myEmitter = new MyEmitter();
var m = 0;
myEmitter.once('event', () => {
  console.log(++m);
});
myEmitter.emit('event');
// Prints: 1
myEmitter.emit('event');
```


## Error event

当 EventEmitter 实例内部出现错误时，通常是发射一个 error 事件，在 Node.js 中，这会作为特殊情况进行处理。
如果 EventEmitter 实例没有注册任何监听函数到 error 事件，当有 error 事件发射时，错误被抛出，打印跟踪栈，Node.js 进程退出。

```javascript
const myEmitter = new MyEmitter();
myEmitter.emit('error', new Error('whoops!'));
// Throws and crashes Node.js
```

为了防止 Node.js 进程崩溃，应该在 process 对象的 uncaughtException 事件上注册监听函数。

```javascript
const myEmitter = new MyEmitter();

process.on('uncaughtException', (err) => {
  console.log('whoops! there was an error');
});

myEmitter.emit('error', new Error('whoops!'));
// Prints: whoops! there was an error
```

最佳实践：总是注册 error 事件的监听函数

```javascript
const myEmitter = new MyEmitter();
myEmitter.on('error', (err) => {
  console.log('whoops! there was an error');
});
myEmitter.emit('error', new Error('whoops!'));
// Prints: whoops! there was an error
```

## EventEmitter 类

事件：

- `'newListener'`
- `'removeListener'`


属性：

- `EventEmitter.defaultMaxListeners`：默认，对任何单一事件最多只能注册 10 个监听函数


### EventEmitter 实例方法


- `emitter.emit(eventName[, ...args])`
- `emitter.addListener(eventName, listener)`：`emitter.emit(eventName[, ...args])` 的别名
- `emitter.eventNames()`：获取 emitter 监听的所有事件名
- `emitter.getMaxListeners()`：获取当前最大可注册监听函数的数量
- `emitter.listenerCount(eventName)`
- `emitter.listeners(eventName)`：返回给定事件的监听函数的数组拷贝
- `emitter.on(eventName, listener)`
- `emitter.once(eventName, listener)`
- `emitter.prependListener(eventName, listener)`
- `emitter.prependOnceListener(eventName, listener)`
- `emitter.removeAllListeners([eventName])`
- `emitter.removeListener(eventName, listener)`
- `emitter.setMaxListeners(n)`

参考资料：

- http://www.infoq.com/cn/articles/tyq-nodejs-event
- https://nodejs.org/api/events.html#events_class_eventemitter




    