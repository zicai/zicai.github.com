---
layout: post
category : lessons
title: "理解 JavaScript 的函数调用和 this"
tagline: "Supporting tagline"
tags : [javascript]
---

JavaScript 函数调用方式以及 `this` 的含义让很多人感到困惑。

在我看来，可以通过理解函数调用的核心原语（primitive），然后把其他调用方式看成核心原语之上的语法糖，来消除这些困惑。事实上，这也是 ECMAScript 规范看待函数调用的方式。在某些方面，本文是规范的简化，但是基本意思是相同的。

## 核心原语 the core primitive

首先，来看函数调用的核心原语，函数的 `call` 方法[1]。


1. 参数中，下标 1 到最后的元素组成参数列表(`argList`)
2. 第一个参数是 `thisValue`
3. 调用函数，把 `this` 设置为 `thisValue`,`argList` 作为参数列表

例如：

```
function hello(thing) {  
  console.log(this + " says hello " + thing);
}

hello.call("Yehuda", "world") //=> Yehuda says hello world 
```

如上所示，调用 `hello` 方法，`this` 设置为 `"Yehuda"` 同时只有一个参数 `"world"`。这就是 JavaScript 函数调用的核心原语。你可以把其他方式脱糖（desugaring）为核心原语。

[1] 在 [ES5 规范中](http://es5.github.com/#x15.3.4.4), `call` 方法是用其它词汇描述的，用更低级的原语，`call` 方法只是对它进行了简单地包装，这里是为了简化，更多信息参见文章末尾。

## 简单函数调用

很明显，每次都要使用 `call` 来调用函数是很烦人的。所以 JavaScript 允许我们用括号语法来直接调用函数 `hellow("world")`。这种调用方式可以脱糖为：

```
function hello(thing) {  
  console.log("Hello " + thing);
}

// this:
hello("world")

// desugars to:
hello.call(window, "world");  
```

在 ECMAScript 5，严格模式下[2]，上面的行为发生了变化：

```
// this:
hello("world")

// desugars to:
hello.call(undefined, "world");  
```

简短的说：函数调用 `fn(...args)` 等同于 `fn.call(window [ES5-strict: undefined], ...args)`。

注意：对 functions declared inline 也是这样 `(function() {})()` 等同于 `(function() {}).call(window [ES5-strict: undefined)`

[2] Actually, I lied a bit. The ECMAScript 5 spec says that undefined is (almost) always passed, but that the function being called should change its thisValue to the global object when not in strict mode. This allows strict mode callers to avoid breaking existing non-strict-mode libraries.

## 成员函数

接下来，另一种常见方式是作为对象的成员函数调用。在这种情况下，脱糖为：

```
var person = {  
  name: "Brendan Eich",
  hello: function(thing) {
    console.log(this + " says hello " + thing);
  }
}

// this:
person.hello("world")

// desugars to this:
person.hello.call(person, "world");  
```

需要注意的是，至于 `hello` 方法是如何附加到对象上的并不会影响上面的行为。上面的例子是初始化对象时定义方法。下面来看，动态的将方法附加到对象上：

```
function hello(thing) {  
  console.log(this + " says hello " + thing);
}

person = { name: "Brendan Eich" }  
person.hello = hello;

person.hello("world") // still desugars to person.hello.call(person, "world")

hello("world") // "[object DOMWindow]world"  
```

需要注意的是：函数并不存在持久的 `this` 值。`this` 的值是在运行时，基于调用方式进行设定的。

## 使用 Function.prototype.bind

有时候，函数具有一个持久的 `this` 值是很方便的。过去，人们经常利用闭包来将一个函数转化为一个 `this` 值不变的函数：

```
var person = {  
  name: "Brendan Eich",
  hello: function(thing) {
    console.log(this.name + " says hello " + thing);
  }
}

var boundHello = function(thing) { return person.hello.call(person, thing); }

boundHello("world");  
```

尽管 `boundHello` 仍然会被脱糖为 `boundHello.call(window, "world")`。但是，我们利用 `call` 方法将 `this` 值改为我们需要的。

为了更加通用，稍作修改：

```
var bind = function(func, thisValue) {  
  return function() {
    return func.apply(thisValue, arguments);
  }
}

var boundHello = bind(person.hello, person);  
boundHello("world") // "Brendan Eich says hello world"  
```

为了理解上面的代码，你需要两点信息：首先，`arguments` 是一个 array-like 对象，代表了传给函数的所有参数。其次，`apply` 方法和 `call` 方法几乎一样，除了它是接受一个 array-like 对象而不是一次性列出所有参数。

我们的 `bind` 方法返回一个新函数。调用它时，新函数会调用原本传入的函数，并按照传入的参数设定 `this` 值。

由于这是很常见的用法，所以ES5 在所有 `Function` 对象上引入了 `bind` 方法，实现了这一行为：

```
var boundHello = person.hello.bind(person);  
boundHello("world") // "Brendan Eich says hello world"  
```

当你需要将一个函数传递做为回调时，`bind` 会更有用:

```
var person = {  
  name: "Alex Russell",
  hello: function() { console.log(this.name + " says hello world"); }
}

$("#some-div").click(person.hello.bind(person));

// when the div is clicked, "Alex Russell says hello world" is printed
```

当然，这样很笨拙，TC39 将会提出更优雅的方案。

## 在 jQuery

由于 jQuery 大量使用了匿名回调函数，在内部，它使用 `call` 方法将这些回调函数的 `this` 值设置为更有用的值。例如：在所有事件处理器中，jQuery 调用 `call` 方法将 `this` 的值设置为绑定事件处理器的元素，而不是接受 `window` 作为 `this` 值。

这非常有用，因为在匿名回调函数中，`this` 的默认值不是特别的有用。

## PS: I Cheated


为了简化，便于理解，上面的用词并不是很准确。Probably the most important cheat is the way I called `func.call` a "primitive". In reality, the spec has a primitive (internally referred to as `[[Call]]`) that both `func.call` and `[obj.]func()`use.

我们来看下 `func.call` 的定义：

1. If IsCallable(func) is false, then throw a TypeError exception.
2. Let argList be an empty List.
3. If this method was called with more than one argument then in left to right order starting with arg1 append each argument as the last element of argList
4. Return the result of calling the [[Call]] internal method of func, providing thisArg as the this value and argList as the list of arguments.




原文地址：[http://yehudakatz.com/2011/08/11/understanding-javascript-function-invocation-and-this/](http://yehudakatz.com/2011/08/11/understanding-javascript-function-invocation-and-this/)
