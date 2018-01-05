---
layout: post
category : node.js
title: "理解 Node.js Buffer"
tagline: "Supporting tagline"
tags : [buffer]
---

## Buffer 简介
### Buffer 的作用
在网络流和文件操作中，需要处理大量的二进制数据。但是在 ES6 引入 TypedArray 之前，JavaScript 语言没有读取和处理二进制流的机制。所以 Node.js API 引入了 Buffer，用来在 TCP 流或是文件系统操作这些场景中处理字节流。

现在 ES6 引入了 TypedArray，Buffer 以一种更适合于 Node.js 使用场景的方式实现了 Unit8Array。

Buffer 类的实例类似于整数数组，但是它对应的是分配在 V8 heap 之外的固定大小的原始内存。Buffer 的大小在创建时就确定了，不可更改。


### Buffer 与字符编码（character encoding）

Buffer 实例常用来表示字符被编码后的字节序列。使用明确的字符编码，就可以在 Buffer 实例和普通 JavaScript 字符串之间相互转换。

例如：

```javascript
const buf = Buffer.from('hello world', 'ascii');

// Prints: 68656c6c6f20776f726c64
console.log(buf.toString('hex'));

// Prints: aGVsbG8gd29ybGQ=
console.log(buf.toString('base64'));
```

目前 Node.js 支持的编码包括：

- `'ascii'`：for 7-bit ASCII data only
- `'utf8'`
- `'utf16le'`
- `'ucs2'`：`'utf16le'` 的别名
- `'base64'`
- `'latin1'`
- `'binary'`：`'latin1'` 的别名
- `'hex'`

对于不支持的编码类型，需要借助第三方模块来完成转码。例如：iconv 和 iconv-lite

### Buffer 与 TypedArray

Buffer 实例同时也是 Uint8Array 实例。但是，与 ECMAScript2015 规范中的 TypedArray 有一些微妙的区别。例如：ArrayBuffer#slice() 会创建一个 slice 的拷贝，而 Buffer#slice() 会在已存在 Buffer 上创建一个 view，而没有拷贝，这使得 Buffer#slice() 更高效。

## 新建 Buffer 实例
Buffer 类在 Node.js 中是全局的，无需 require

之前是通过构造函数创建新实例，但是在 Node.js v6.0 中将 `new Buffer()` 标记为 deprecated。所以现在新建 Buffer 实例，要用以下几种方式：

- `Buffer.alloc()`
- `Buffer.allocUnsafe()`
- `Buffer.allocUnsafeSlow()`
- `Buffer.from()`

### Buffer.alloc()

```
Buffer.alloc(size[, fill[, encoding]])
```
其中：

- size 是新 Buffer 的长度
- fill 可以是字符串、Buffer 或是整数。用来预填充新 Buffer。默认是 0。
- encoding 当 fill 是字符串时，用来设置编码。默认是 'utf8'

如果指定了 fill，分配的 Buffer 就会通过调用 `buf.fill(fill)` 来初始化。如果同时也指定了 encoding，则调用 `buf.fill(fill, encoding)`。

调用 `Buffer.alloc()` 会比调用下面的 `Buffer.allocUnsafe()` 明显慢很多，但是新创建的 Buffer 对象中不会包含敏感数据。

### Buffer.allocUnsafe()

```
Buffer.allocUnsafe(size)
```

用这种方式创建的 Buffer 实例使用的底层内存没有进行初始化。新 Buffer 实例的内容不可知，可能会包含敏感数据。

### Buffer.allocUnsafeSlow()

```
Buffer.allocUnsafeSlow(size)
```

### Buffer.from()

```
Buffer.from(array)
Buffer.from(arrayBuffer[, byteOffset[, length]])
Buffer.from(buffer)
Buffer.from(string[, encoding])
```

## Buffer 类的方法和属性

- `Buffer.byteLength(string[, encoding])` 获取字符串实际的字节长度。和 `String.prototype.length` 不同，后者返回的是字符数。
- `Buffer.compare(buf1, buf2)`：比较两个 Buffer，通常是为了对 buffer 实例组成的数组进行排序。等同于 `buf1.compare(buf2)`
- `Buffer.concat(list[, totalLength])`：返回一个新的 Buffer，它由 list 中所有 Buffer 实例拼接而成。
- `Buffer.isBuffer(obj)`：如果 obj 是一个 Buffer，则返回 true，否则返回 false
- `Buffer.isEncoding(encoding)`：判断是否支持某种编码

### Buffer 类的属性

- `Buffer.poolSize`


## 操作 buffer 实例

buffer 实例是一个类似数组的对象，可以访问 length 属性，如：`buf.length`，也可以通过下标访问元素，如：`buf[index]`

### buffer 比较
 
- `buf.compare(target[, targetStart[, targetEnd[, sourceStart[, sourceEnd]]]])`
- `buf.equals(otherBuffer)`：判断 buffer 是否相等

### 填充与复制 buffer

- `buf.fill(value[, offset[, end]][, encoding])`：填充 buffer
- `buf.copy(target[, targetStart[, sourceStart[, sourceEnd]]])`：从一个 buffer 拷贝数据到另一个 buffer
- `buf.slice([start[, end]])`：返回一个新的 buffer 实例，和原来的 buffer 引用相同的内存，偏移由 start 和 end 决定。二者是有一部分是重叠的，修改一个，另一个也会有变化。


### 在 buffer 中查找

- `buf.indexOf(value[, byteOffset][, encoding])`：
- `buf.lastIndexOf(value[, byteOffset][, encoding])`
- `buf.includes(value[, byteOffset][, encoding])`

### 遍历
buffer 实例可以用 ES6 的 `for...of` 语法进行遍历。

```
const buf = Buffer.from([1, 2, 3]);

// Prints:
//   1
//   2
//   3
for (const b of buf) {
  console.log(b);
}
```
下面三个方法用来创建遍历器（iterator）

- `buf.values()`
- `buf.keys()`
- `buf.entries()`


### 交换字节序（byte-order）

* `buf.swap16()`
* `buf.swap32()`
* `buf.swap64()`

### 格式转换

* `buf.toJSON()`：返回 buffer 的 JSON 表示。当对 buffer 实例使用 JSON.stringfy() 时，实际上会隐式的调用该方法
* `buf.toString([encoding[, start[, end]]])`：将 buf 按照 encoding 解码成字符串。



### buffer 读

数字的编码要考虑三个不同属性：

- 数字的 bit-length，也就是一个数字需要多少个位来表示
- 数字类型：
	- 有符号还是无符号
	- 在 Node 中，数字可以是 integer、float 和 double。float 和 double 都是小数，区别是 float bit-length 为 32，而 double 是 64.
- 端(Endian)：解析字节时，是从左向右还是从右向左。

所以从 buffer 中读取数字时，要选择对应的方法：BE 和 LE 分别表示大端法（big-endian） 和小端法（little-endian）。注意：8位（bit）整数是没有大端和小端区分，因为端是字节级别的属性，8位只是一个字节。


* buf.readIntBE(offset, byteLength[, noAssert])
* buf.readIntLE(offset, byteLength[, noAssert])
* buf.readUIntBE(offset, byteLength[, noAssert])
* buf.readUIntLE(offset, byteLength[, noAssert])
* buf.readInt8(offset[, noAssert])
* buf.readUInt8(offset[, noAssert])
* buf.readInt16BE(offset[, noAssert])
* buf.readInt16LE(offset[, noAssert])
* buf.readUInt16BE(offset[, noAssert])
* buf.readUInt16LE(offset[, noAssert])
* buf.readInt32BE(offset[, noAssert])
* buf.readInt32LE(offset[, noAssert])
* buf.readUInt32BE(offset[, noAssert])
* buf.readUInt32LE(offset[, noAssert])
* buf.readDoubleBE(offset[, noAssert])
* buf.readDoubleLE(offset[, noAssert])
* buf.readFloatBE(offset[, noAssert])
* buf.readFloatLE(offset[, noAssert])

### buffer 写

- `buf.write(string[, offset[, length]][, encoding])`：将 string 写入 buffer

写入数字时，要选择合适的方法：

* buf.writeDoubleBE(value, offset[, noAssert])
* buf.writeDoubleLE(value, offset[, noAssert])
* buf.writeFloatBE(value, offset[, noAssert])
* buf.writeFloatLE(value, offset[, noAssert])
* buf.writeInt8(value, offset[, noAssert])
* buf.writeInt16BE(value, offset[, noAssert])
* buf.writeInt16LE(value, offset[, noAssert])
* buf.writeInt32BE(value, offset[, noAssert])
* buf.writeInt32LE(value, offset[, noAssert])
* buf.writeIntBE(value, offset, byteLength[, noAssert])
* buf.writeIntLE(value, offset, byteLength[, noAssert])
* buf.writeUInt8(value, offset[, noAssert])
* buf.writeUInt16BE(value, offset[, noAssert])
* buf.writeUInt16LE(value, offset[, noAssert])
* buf.writeUInt32BE(value, offset[, noAssert])
* buf.writeUInt32LE(value, offset[, noAssert])
* buf.writeUIntBE(value, offset, byteLength[, noAssert])
* buf.writeUIntLE(value, offset, byteLength[, noAssert])

## Buffer 拼接
Buffer 在使用场景中，通常是以一段一段的方式传输。在拼接过程中经常会出现问题：

```
data += chunk
```

上面这句代码隐藏了 `toString()` 操作，等价于：

```
data = data.toString() + chunk.toString()
```

可以通过可读流的 setEncoding 方法来解决：

```
readable.setEncoding(encoding)
```

该方法的作用是让 data 事件中传递的不再是 Buffer 对象，而是编码后的字符串。原理是其内部神奇的 StringDecoder 实例。但它并非万能的，只能处理 UTF-8、base64和 UCS-2/UTF-16LE  这三种编码。

要想从根本上解决该问题就得正确拼接 Buffer，然后通过 iconv 一类的模块来转码。不能用 += 的方式拼接，需要用到 `Buffer.concat()` 方法。

假设有 GBK 编码的网页内容
```javascript
http.request({
	hostname:'http://www.google.com'
},(res) => {
	let bufArray = [];
	let bytesLen = 0;
	res.on('data',(chunk) => {
		bufArray.push(chunk);
		bytesLen += chunk.length;
	});
	res.on('end',() => {
		let bufContent = Buffer.concat(bufArray, bytesLen);
		let strContent = iconv.decode(bufContent,'gbk')
	})
})
```



    