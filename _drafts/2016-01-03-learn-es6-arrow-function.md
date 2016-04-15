---
layout: post
category : lessons
title: "学习ES6--箭头函数"
tagline: "Supporting tagline"
tags : [javascript]
---

ES6 引入箭头函数（Fat arrow function 或是 arrow function）有两个目的：

- 更简洁的语法
- sharing lexical this with the parent scope


## 与普通函数的区别

- Lexical this binding – The value of this inside of the function is determined by where the arrow function is defined not where it is used.
- Not newable – Arrow functions cannot be used a constructors and will throw an error when used with new.
- Can’t change this – The value of this inside of the function can’t be changed, it remains the same value throughout the entire lifecycle of the function.




## 新的函数语法

先看旧的语法，以一个Promise chain 为例：

```
function getVerifiedToken(selector) {
  return getUsers(selector)
    .then(function (users) { return users[0]; })
    .then(verifyUser)
    .then(function (user, verifiedToken) { return verifiedToken; })
    .catch(function (err) { log(err.stack); });
}
```

用箭头函数重写上面代码：

```
function getVerifiedToken(selector) {
  return getUsers(selector)
    .then(users => users[0])
    .then(verifyUser)
    .then((user, verifiedToken) => verifiedToken)
    .catch(err => log(err.stack));
}
```

有几点需要注意：

- 当回调函数的逻辑代码只有一行时，可以省略了`function` 和 `{}`
- 只有一个参数时可以省略 `()`。(rest arguments are an exception, eg (...args) => ...)
- 省略了 `return` 关键字，是因为当省略 `{}` 时，单行声明箭头函数隐式 return(这种函数在其它语言里通常称为lambda functions)

需要再次强调上面的最后一点。当箭头函数声明时包含了 `{}`，即使它是单行声明，隐式return 也不会发生。

When we talk about single statement arrow functions, it doesn’t mean the statement can’t be spread out to multiple lines for better comprehension.

警告：当省略`{}`时，并需要返回对象时，需要用`()`，例如：`({})`。

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

JavaScript 中的 this 饱受诟病。以jQuery 为例，下面的代码每分钟更新时间

```
$('.current-time').each(function () {
  setInterval(function () {
    $(this).text(Date.now());
  }, 1000);
});
```

当在 setInterval 中尝试通过each 设定的 this 引用DOM元素时，我们实际上得到的是属于回调函数本身的一个this。常见的解决办法是：

```
$('.current-time').each(function () {
  var self = this;
 
  setInterval(function () {
    $(self).text(Date.now());
  }, 1000);
});
```

使用箭头函数就不存在这个问题，因为它们并不引入自己的this:

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

重申一下：箭头函数没有自己的 this 和 `arguments`。但是，你还是可以通过 rest parameters(also known as spread operator)来获取传入箭头函数的所有参数。

```
function log(msg) {
  const print = (...args) => console.log(args[0]);
  print(`LOG: ${msg}`);
}
 
log('hello'); // LOG: hello
```

## 关于 yield

箭头函数不能用作generator。

## 底线
不建议用箭头函数改写所有函数。建议需要用到下面的新特性时，再使用箭头函数：

- 立即返回的单行声明函数（lambdas）
- 需要使用父scope的this 的函数


参考链接：


- https://www.nczonline.net/blog/2013/09/10/understanding-ecmascript-6-arrow-functions/
- https://strongloop.com/strongblog/an-introduction-to-javascript-es6-arrow-functions/
























