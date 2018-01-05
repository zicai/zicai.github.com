---
layout: post
category : node.js
title: "理解 Node.js stream"
tagline: "Supporting tagline"
tags : [stream]
---

stream 是一个用来处理数据流的抽象接口。

Node.js 自身也提供了许多 stream 对象。例如：到 HTTP server 的请求和 `process.stdout` 都是 stream 实例。

stream 可以是可读、可写或是都可以。所有流都是 EventEmitter 的实例。

使用 stream 模块：

```js
const stream = require('stream');
```

如果你主要是使用 stream 对象，那么应该几乎用不到 stream 模块，stream 模块主要是给那些需要创建新的 stream 实例类型的开发者使用的。该模块提供了基础的 API，使得创建对象实现 stream 接口变得很容易。


## 流的类型

- Readable：可以从中读取数据（例如：`fs.createReadStream()`）
- Writable：可以写数据（例如：`fs.createWriteStream()`）
- Duplex：既可写又可读（例如：`net.socket`）
- Transform：在读或写的过程中可以变换数据的 Duplex 流（例如：`zlib.createDeflate()`）


## Object Mode

## Buffering

可读流和可写流都会把数据保存在一个内部 buffer 里，对应的，分别可以通过 `readable._readableState.buffer` 和 `writable._writableState.getBuffer()` 来访问。

传给 stream 构造函数的 `highWaterMark` 参数决定了可能缓冲的数据量。对常规流来说，`highWaterMark` 指定的是字节数。对处于 Object mode 的流，`highWaterMark` 指定的是对象数。

当调用 `stream.push(chunk)` 时，数据被缓冲到可读流中。如果流的消费者没有调用 `stream.read()`，那么数据会保持在内部队列中直到被消费。

一旦内部缓冲的数据达到 `highWaterMark` 指定的限度时，stream 就会暂时停止从底层资源读取数据直到当前缓冲的数据可以被消费（也就是说，stream 停止调用内部方法 `readable._read()`，它用来填充 read buffer）

当重复调用 `writable.write(chunk)` 方法时，数据被缓冲在可写流中。当内部缓冲的数据量小于 `highWaterMark` 指定的限度时，`writable.write()` 返回 true，当数据量达到或者超过限度时，返回 false。

stream API 一个主要目的就是 `stream.pipe()` 方法，把缓冲的数据量限制在一个可接受范围，这样不同速率的输入源和输出不至于把内存全吃掉。

由于 Duplex 和 Transform 流都是既可读又可写的，所以它们都维持两个独立的内部 buffer 分别用来读和写，在保证效率的同时又互不影响。


## 流的消费者

### 可写流
可写流是对数据流向的目的地的抽象。所有可写流都实现了 stream.Writable 类定义的接口。虽然特定的可写流的实例可能略有不同，但基本使用模式是一致的：

```js
const myStream = getWritableStreamSomehow();
myStream.write('some data');
myStream.write('some more data');
myStream.end('done writing data');
```

stream.Writable 类的包含的事件

- `'close'`：当流或者底层资源（例如：文件描述符）关闭时，发射 `'close'` 事件。该事件表示不会再发射别的事件了。不是所有可写流都会发射 `'close'` 事件。
- `'drain'`：如果调用 `writable.write(chunk)` 返回 false（缓冲区已满），当流再次可以写入时，发射 `'drain'`。
- `'error'`：当写数据或者 piping 数据发生错误时，发射 `'error'` 事件。监听函数被调用时会接收到一个 Error 对象。当 `'error'` 事件发生时，stream 并不会被关闭。
- `'finish'`：调用 stream.end() 之后会发射 `'finish'` 事件，并且所有数据都被 flush 到底层系统。
- `'pipe'`：在可读流上调用 stream.pipe() 方法时，发射 `'pipe'` 事件
- `'unpipe'`：在可读流上调用 stream.unpipe() 方法时，发射 `'unpipe'` 事件

方法

- `writable.cork()`
- `writable.uncork()`
- `writable.write(chunk[, encoding][, callback])`
- `writable.end([chunk][, encoding][, callback])`：表示再没有数据要写入了
- `writable.setDefaultEncoding(encoding)`

### 可读流


#### 两种模式
Readable stream 有两种模式：

- flowing mode：数据尽可能快的自动从底层系统读取，传输到你的程序中。
- paused mode：必须显式的调用 `stream.read()` 来把数据读出来。

Stream 以 paused mode 开始，然后可以用下面的方式在两种模式间切换：

可以通过下面方式切换到 flowing mode:

- 添加 `'data'` 事件监听器
- 调用 `stream.resume()` 方法显式的打开 flow
- 调用 `stream.pipe()` 方法，发送数据到可写流

可以通过下面方式切换到 paused mode:

- 如果不存在 pipe 目标，通过调用 `stream.pause()` 方法
- 如果存在 pipe 目标，要移除所有 `'data'` 事件监听器 ，而且要通过 `stream.unpipe()` 方法移除所有 pipe 目标。

需要注意的是：为了兼容旧版本，移除所有 `'data'` 事件不会自动暂停流。而且，如果存在 pipe 目标，调用 `stream.pause()` 也不能保证流停在 pause 状态，因为目标仍有可能索取数据。

注意：如果可读流切换到 flowing mode，但是没有对应的消费者来处理数据，数据就会丢失。例如：没有附加 `'data'` 事件监听器，也没有 `stream.pipe()` 目标，当流变为 flow mode，则数据会流失。

#### 三种状态

两种模式实际上是对可读流内部更复杂的状态管理的简单抽象。

在任意给定时刻，可读流处于下面三种状态之一：

- `readable._readableState.flowing = null`
- `readable._readableState.flowing = false`
- `readable._readableState.flowing = true`

#### 选一种

在 Node.js 中，提供了多种消费流数据的方法。开发者应该选取其中一种，永远不要使用多个方法从单一流中消费数据。

对大多数用户而言，建议使用 `readable.pipe()`，它是最简单的方式。如果想要细粒度的控制数据的传输和生成可以使用 EventEmitter 和 `readable.pause()` / `readable.resume()` API。

事件：

- `'close'`：当流或者底层资源（例如：文件描述符）关闭时，发射 `'close'` 事件。该事件表示不会再发射别的事件了。不是所有可读流都会发射 `'close'` 事件。
- `'data'`：当流放弃一段数据的所有权，交给消费者时触发。发生在通过调用 `readable.pipe()`，`readable.resume()` 或是添加监听函数到 'data' 事件将流切换到 flowing mode 时触发。无论何时调用 `readable.read()` 也会发射 `'data'` 事件。
- `'end'`：没有数据可供消费时触发。注意：除非数据被完全被消费，否则不会发射 `'end'` 事件。
- `'error'`：
- `'readable'`：当数据可读时发射 `'readable'` 事件

方法：

- `readable.isPaused()`：判断可读流的运行状态。主要是给 readable.pipe() 方法的底层机制使用，通常不会直接用到
- `readable.pause()`
- `readable.pipe(destination[, options])`：返回目标流的引用，这样就可以串联多个 `pipe()` 方法。
- `readable.read([size])`：默认，读出来的数据时 Buffer 对象，除非用 readable.setEncoding() 指定了编码或者流运行在 object mode。如果没有指定 size，那么就返回内部 buffer 包含的所有数据。只应该在可读流处在暂停模式时才使用 readable.read() 方法。在 flowing 模式，readable.read() 会自动被调用直到内部 buffer 为空。
- `readable.resume()`
- `readable.setEncoding(encoding)`
- `readable.unpipe([destination])`：如果没有指定目标，则绑定的所有 pipe 都被解绑。
- `readable.unshift(chunk)`：将一块数据退回到内部 buffer
- `readable.wrap(stream)`

### Duplex 和 Transform 流
Duplex 流的例子：

- TCP sockets
- zlib streams
- crypto streams

Transform 流的例子：

- zlib streams
- crypto streams
    
## 流的实现

要实现任何形式的流，模式是相同的：

1. 在你的子类中继承适当的父类
2. 在你的构造函数中调用适当父类的构造函数
3. 实现一个或多个特定方法

根据使用场景不同，继承不同的父类，实现对应的方法：

|使用场景|类|要实现的方法
|---|:----|:---|
|只读|Readable|_read|
|只写|Writable|_write,_writev|
|可读可写|Duplex|_read,_write,_writev|
|操作写入的数据，然后读取操作结果|Transform|_transform,_flush|

在你的实现代码中，绝对不要调用流的消费者的方法，因为可能产生不良影响。

## 为什么使用流

Streams make programming in node simple, elegant, and composable.

## 基础

存在五种流：readable, writable, transform, duplex, 和 "classic".

### pipe()

### readable streams

#### 创建 readable streams

#### 消耗 readable streams

### writable streams

#### 创建 writable streams
#### 写入到 writable streams

### transform

### duplex

### classic streams

## 内置 流

## 控制 流

- [through](https://github.com/dominictarr/through)--simple way to create a ReadableWritable stream that works
- [from](https://github.com/dominictarr/from)--Easy way to create a Readable Stream
- pause-stream
- [concat-stream](https://github.com/maxogden/concat-stream)--writable stream that concatenates strings or data and calls a callback with the result
- 

## meta streams

## state treams

## http 流

## io 流

## parser streams

## browser streams

## html streams

## audio streams

## rpc streams

## test streams

## 强大的组合

参考链接：

- https://nodejs.org/api/stream.html
- [node.js stream-handbook](https://github.com/substack/stream-handbook)
- stream-handbook 完整中文版 https://github.com/jabez128/stream-handbook
- http://tech.meituan.com/stream-basics.html
