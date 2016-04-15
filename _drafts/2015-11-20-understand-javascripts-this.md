---
layout: post
category : lessons
title: "理解 JavaScript 的 this"
tagline: "Supporting tagline"
tags : [javascript]
---

The ECMAScript Standard defines this as a keyword that "evaluates to the value of the ThisBinding of the current execution context" (§11.1.1). ThisBinding is something that the JavaScript interpreter maintains as it evaluates JavaScript code, like a special CPU register which holds a reference to an object. The interpreter updates the ThisBinding whenever establishing an execution context in one of only three different cases:

大多数情况下，this的值取决于函数的调用方法。在执行过程中不能通过赋值给this，函数每次调用的this值都可能不同。
ES5 引入了bind 方法，设定函数的this值而不用管它是如何被调用的。
ES6 引入了箭头函数，。。。。

## global context
global execution context（outside of any function）,this 引用的是全局对象。无论是否是严格模式。

## function context
在函数里，this的值取决于函数如何调用

### 简单调用

### 箭头函数

### 对象的方法

### 构造函数

### call 和 apply

### bind

### DOM event handler

### in-line event handler

## eval
eval() either picks up the current value of this or sets it to the global object, depending on whether it is called directly or indirectly.


参考资料：

- https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this
- DOM 事件绑定中的 this http://www.quirksmode.org/js/this.html
- http://www.2ality.com/2014/05/this.html
- http://stackoverflow.com/questions/3127429/how-does-the-this-keyword-work