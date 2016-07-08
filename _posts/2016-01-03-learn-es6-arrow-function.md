---
layout: post
category : lessons
title: "学习ES6--箭头函数"
tagline: "Supporting tagline"
tags : [javascript]
---


## 箭头函数有何作用
ES6 引入箭头函数（Fat arrow function 或是 arrow function）出于两个目的：

- 更简洁的语法
- sharing lexical this with the parent scope


## 与普通函数的异同

不同点：

- Lexical this binding – 箭头函数内部 `this` 值由其定义的位置决定，而不是使用的位置。
- Not newable – 箭头函数不能用作构造函数，和 `new` 一起使用会抛出错误。
- Can’t change this – 箭头函数内部 `this` 值不可改变，在函数整个生命周期中都是同一个值。


相同点：

- `typeof` 箭头函数返回 `'function'`
- 箭头函数也是 `Function` 的实例，所以 `instanceof` 结果相同。
- 可以在箭头函数上使用 `call()`,`apply()` 和 `bind()`。但是 `this` 值不可变。

## 语法

箭头函数的语法灵活多变。基本结构是：函数参数，跟着一个箭头，跟着函数体。例如：

```
var reflect = value => value;

// 等同于:

var reflect = function(value) {
    return value;
};
```

如果不止一个参数，则需要用括号括起来，如下：

```
var sum = (num1, num2) => num1 + num2;

// 等同于:

var sum = function(num1, num2) {
    return num1 + num2;
};
```

如果不需要参数，也得用括号括起来，如下：

```
var sum = () => 1 + 2;

// 等同于:

var sum = function() {
    return 1 + 2;
};
```

因为花括号用来表示函数体，箭头函数如果要返回一个 literal object，必须用括号括起来。如下：

```
var getTempItem = id => ({ id: id, name: "Temp" });

// 等同于:

var getTempItem = function(id) {

    return {
        id: id,
        name: "Temp"
    };
};
```


有几点需要注意：

- 只有一个参数时可以省略 `()`，rest arguments 除外，例如： `(...args) => ...`
- 当省略 `{}` 时，单行箭头函数会隐式 return (这种函数在其它语言里通常称为 lambda functions)

需要再次强调上面的最后一点。当箭头函数声明时包含了 `{}`，即使它是单行声明，隐式 return 也不会发生。


综上所述：

```
function () { return 1; }
() => { return 1; }
() => 1
 
function (a) { return a * 2; }
(a) => { return a * 2; }
(a) => a * 2
a => a * 2
 
function (a, b) { return a * b; }
(a, b) => { return a * b; }
(a, b) => a * b
 
function () { return arguments[0]; }
(...args) => args[0]
 
() => {} // undefined
() => ({}) // {}
```

## Lexical this

JavaScript 中的 `this` 饱受诟病。以 jQuery 为例，下面的代码用于更新时间：

```
$('.current-time').each(function () {
  setInterval(function () {
    $(this).text(Date.now());
  }, 1000);
});
```

当在 `setInterval` 中尝试通过 `each` 设定的 `this` 引用 DOM 元素时，我们实际上得到的是属于回调函数本身的一个 `this`。常见的解决办法是：

```
$('.current-time').each(function () {
  var self = this;
 
  setInterval(function () {
    $(self).text(Date.now());
  }, 1000);
});
```

或者使用 `.bind()`

```
$('.current-time').each(function () {
  setInterval((function () {
    $(this).text(Date.now());
  }).bind(this), 1000);
});
```


使用箭头函数就不存在这个问题，因为它们并不引入自己的 `this`:

```
$('.current-time').each(function () {
  setInterval(() => $(this).text(Date.now()), 1000);
});
```

## 关于参数
警告：箭头函数没有 `arguments` 变量

```
function log(msg) {
  const print = () => console.log(arguments[0]);
  print(`LOG: ${msg}`);
}
 
log('hello'); // hello
```

重申一下：箭头函数没有自己的 `this` 和 `arguments`。但是，你还是可以通过 rest parameters 来获取传入箭头函数的所有参数。

```
function log(msg) {
  const print = (...args) => console.log(args[0]);
  print(`LOG: ${msg}`);
}
 
log('hello'); // LOG: hello
```

## 关于 yield

箭头函数不能用作 generator。


## 底线
不建议用箭头函数改写所有函数。建议需要用到下面的新特性时，再使用箭头函数：

- 立即返回的单行声明函数（lambdas）
- 需要使用父 scope 的 `this` 的函数


参考链接：


- https://www.nczonline.net/blog/2013/09/10/understanding-ecmascript-6-arrow-functions/
- http://exploringjs.com/es6/ch_arrow-functions.html
- https://strongloop.com/strongblog/an-introduction-to-javascript-es6-arrow-functions/
- http://tc39wiki.calculist.org/es6/arrow-functions/
- http://www.2ality.com/2016/02/arrow-functions-vs-bind.html
