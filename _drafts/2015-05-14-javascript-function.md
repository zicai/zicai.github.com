---
layout: post
category : lessons
title: "javascript 函数总结"
tagline: "Supporting tagline"
tags : [javascript]
---

函数也是对象。和普通对象的区别在于函数对象内部有一个名叫 `[[Call]]` 属性。表明该对象是可以执行的。因为只有函数对象有这个属性，所以对于任何包含 `[[Call]]` 属性的对象， `typeof` 操作符都返回 `"function"`。

## 定义函数

定义函数有两种语法：

- 函数声明：

	```
	function sum(num1,num2){
		return sum1+sum2;
	}
	```
- 函数表达式：

	```
	var sum=function(num1,num2){
		return sum1+sum2;
	}
	```
	
`return` 语句之后的任何代码永远都不会执行。`return` 语句也可以不带有任何返回值，在这种情况下，函数在停止执行后将返回 `undefined`。

在代码开始运行之前，解析器会通过一个名为函数声明提升(function declaration hoisting)的过程，使其在执行任何代码之前可用；至于函数表达式，则必须等到解析器执行到它所在的代码行，才会真正的被解释执行。


## 函数的参数

ECMAScript 函数不在乎传进来参数的数量和类型。因为 ECMAScript 中参数在内部是用一个类数组来表示的，函数接收到的始终都是这个数组。

在函数内部通过 arguments 对象来访问这个数组。命名参数只是为了便于访问。

没有传递值的命名参数会被赋予 undefined，类似于定义了变量却没有初始化。

ECMAScript 中的所有参数传递的都是值，不可能通过引用传递参数。

## 重载

大多数面向对象语言都支持函数重载（function overloading），即一个函数有多个签名。函数签名由函数名字和参数数量和类型组成。语言会根据传入的参数调用对应的函数。

由于不存在函数签名的特性，ECMAScript 函数不能重载。


## 函数内部属性

在函数内部，有两个特殊的对象：`arguments`和`this`

`arguments`的主要用途是保存函数参数，它还有一个`callee`属性，指向拥有这个`arguments`对象的函数。

`this`引用的是函数据以执行的环境对象。

## 函数的属性和方法

每个函数包含两个属性：`length`和`prototype`。其中`length`属性表示函数希望接收的命名参数的个数。

`prototype`是保存它们所有实例方法的真正所在。`prototype`属性是不可枚举的，因此使用for-in 无法发现。

ECMAScript 5规范了另一个函数对象的属性：`caller`。这个属性保存着调用当前函数的的函数的引用。

每个函数都包含两个非继承而来的方法：`apply()`和`call()`。它俩的用途是在特定的作用域中调用函数，实际上等于设置函数体内`this`对象的值。


`apply()`方法接收两个参数：一个是在其中运行函数的作用域，另一个是参数数组，可以是`Array`的实例，也可以是`arguments`对象。

`call()`方法与`apply()`方法区别仅在于接收参数的方式不同。使用`call()`方法时，传递给函数的参数必须逐个列举出来。

使用`call()`和`apply()`来扩充作用域最大的好处是，对象不需要与方法有任何耦合关系。

ECMAScript 5还定义了一个方法`bind()`。这个方法会创建一个函数的实例，其`this`的值会被绑定到传给`bind()`函数的值。例如：

```
var o={color:'red'};

function sayColor(){
	alert(this.color);
}

var objectSayColor=sayColor.bind(o);

objectSayColor(); //red
```

参考资料：[https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)

