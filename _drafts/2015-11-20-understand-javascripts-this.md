---
layout: post
category : lessons
title: "理解 JavaScript 的 this"
tagline: "Supporting tagline"
tags : [javascript]
---

在 Java、PHP 这样的语言中，类方法中的 `this` 被看做当前对象的实例。它不能在方法之外使用，简单没有歧义。

在 JavaScript 中，`this` 是一个函数当前的执行上下文。由于 JS 有四种调用函数的方式：

- 函数调用：`alert('Hello World!')`
- 方法调用：`console.log('Hello World!')`
- 构造函数调用：`new RegExp('\\d')`
- 非直接调用：`alert.call(undefined, 'Hello World')`

每一种调用方式对应不同的上下文。

而且 strict mode 也会影响执行上下文。

理解 `this` 的关键点在于要对函数调用如何影响上下文有一个清晰地认识。

本文专注于解释函数调用，以及函数调用如何影响 `this`，和确定上下文时常见的误区。

开始之前，先来熟悉一些词汇：

- Invocation
- Context
- Scope


## 函数调用
### 函数调用中的 this 值

> 在函数调用中 this 值是全局对象

全局对象由执行环境决定。在浏览器中是 window 对象。

在全局执行上下文中（global execution context（outside of any function）），this 引用的也是全局对象。

### 严格模式下，函数调用中的 this 值


> 严格模式下，函数调用中的 this 值为 undefined

ECMAScript 5.1 引入了严格模式。
 
严格模式不仅会影响当前 scope，同时会影响 inner scope（所有内部声明的函数）。
 
 
### 误区：内部函数的 this 值
 
一个常见的陷阱是：认为内部函数的 this 值与外部函数一样。实际上，内部函数的上下文只取决于调用方式，而非外部函数的上下文。

为了获得正确的 this 值，通常需要使用 .call() .apply() 或 .bind()

## 方法调用

### 方法调用中的 this 值

> 方法调用中 this 值为包含该方法的对象

JavaScript 对象可以从 prototype 中继承方法。当调用这些继承的方法时，调用的上下文仍然是对象本身。

在 ECMAScript 6 class 语法中，方法调用的上下文也是实例本身。

### 误区：从对象上分离方法

对象的方法可以赋值给一个独立的变量。当通过这个变量调用该方法时，你可能会认为 this 值为定义方法的对象。实际上，如果方法没有通过对象调用，就成了函数调用。那么，this 值就是全局对象 window 或是严格模式下的 undefined。

```
function Animal(type, legs) {  
  this.type = type;
  this.legs = legs;  
  this.logInfo = function() {
    console.log(this === myCat); // => false
    console.log('The ' + this.type + ' has ' + this.legs + ' legs');
  }
}
var myCat = new Animal('Cat', 4);  
// logs "The undefined has undefined legs"
// or throws a TypeError in strict mode
setTimeout(myCat.logInfo, 1000);  
```

在上面的例子中，当方法作为参数传递时，就从对象上剥离了。使用 bind() 可以修正该行为：

```
function Animal(type, legs) {  
  this.type = type;
  this.legs = legs;  
  this.logInfo = function() {
    console.log(this === myCat); // => true
    console.log('The ' + this.type + ' has ' + this.legs + ' legs');
  };
}
var myCat = new Animal('Cat', 4);  
// logs "The Cat has 4 legs"
setTimeout(myCat.logInfo.bind(myCat), 1000);  
```

## 构造函数调用

如果构造函数调用时没有参数，那么括号可以省略。

从 ECMAScript 6 开始，可以使用 class 关键字定义构造器。

## 非直接调用
## Bound function

## 箭头函数

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

_ http://rainsoft.io/gentle-explanation-of-this-in-javascript
- https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this
- DOM 事件绑定中的 this http://www.quirksmode.org/js/this.html
- http://www.2ality.com/2014/05/this.html
- http://stackoverflow.com/questions/3127429/how-does-the-this-keyword-work