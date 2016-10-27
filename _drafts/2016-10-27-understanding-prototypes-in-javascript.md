---
layout: post
category : javascript
title: "理解 JavaScript 的原型对象和原型链"
tagline: "Supporting tagline"
tags : [prototype]
---

ECMAScript 支持面向对象编程，但是和传统面向对象语言（例如：Java）不同的是，ECMAScript 不使用类或者接口，而是利用对象和原型对象。甚至可以通过构造原型链，来实现继承的效果。

所以，理解 JavaScript 面向对象编程的关键就在于理解原型对象和原型链。

### 背景知识

- 函数也是对象，它可以有自己的属性
- JavaScript 中对象的属性分为三种：
	- 数据属性（Named data properties）
	- 访问器属性（Named accessor properties）
	- 内部属性（Internal properties）：这些属性只在规范中使用，不能通过脚本直接访问。通常表示为[[propertyName]]


## 原型对象

首先呢，原型对象的作用是，它作为容器，为所有实例提供共享的属性和方法。

接下来，仔细阅读下面三句话：

1. 无论什么时候，只要创建了一个新的函数，就会对应生成一个原型对象，同时会给该函数创建一个 `prototype` 属性（因为函数本身也是一个对象），这个属性指向函数的原型对象。
2. 默认情况下，所有原型对象都会包含一个 `constructor` 属性，这个属性包含一个指针，指向 `prototype` 属性所在的函数。
3. 当调用构造函数创建一个新实例后，该实例内部将包含一个指针（内部属性，表示为`[[Prototype]]`），它指向构造函数的原型对象。

注意：注意区分函数的 `prototype` 属性和实例的内部属性 `[[Prototype]]`。

通过代码来理解上面三句话：

```
function Person(){

}

console.log(typeof Person.prototype) // 输出 "object"
console.log(Person.prototype.constructor === Person) // 输出 true

var person1 = new Person()
```

新建了一个 `Person` 函数，会对应生成一个原型对象，`Person` 函数会包含一个 `prototype` 属性，该属性指向函数的原型对象。原型对象会包含一个 `constructor` 属性，该属性指向 `Person` 函数本身。

它们之间的关系可以用下图表示：

![](http://ofpfd31sq.bkt.clouddn.com/%E7%90%86%E8%A7%A3%20JavaScript%20%E7%9A%84%E5%8E%9F%E5%9E%8B%E5%AF%B9%E8%B1%A1%E5%92%8C%E5%8E%9F%E5%9E%8B%E9%93%BE-%E5%9B%BE1.svg)

注意：实例和构造函数之间没有直接的关系，这个连接存在于实例与构造函数的原型对象之间。

## 原型链

上面的代码中，`Person` 函数对应的是默认的原型对象，
想一想，如果我们重写函数的默认原型对象，让原型对象等于另一个类型的实例，会怎样呢？

```
function SuperType() {

}


function SubType() {

}

SubType.prototype = new SuperType();

var instance = new SubType();
```
在上面的代码中，我们重写了 `SubType` 函数默认的原型对象，将其设置为 `SuperType` 的一个实例。此时，`SubType` 的实例 `instance` ，它的 `[[prototype]]` 属性指向的是 `SuperType` 的一个实例。而这个实例，同样包含一个 `[[prototype]]` 属性，指向的是 `SuperType` 函数的原型对象。

它们之间的关系如下图所示：

![](http://ofpfd31sq.bkt.clouddn.com/%E7%90%86%E8%A7%A3%20JavaScript%20%E7%9A%84%E5%8E%9F%E5%9E%8B%E5%AF%B9%E8%B1%A1%E5%92%8C%E5%8E%9F%E5%9E%8B%E9%93%BE-%E5%9B%BE2.svg)

依照上面的模式，我们就可以构造出一个原型链。

### 原型链的作用

要知道，访问对象的某个属性时，会沿着原型链执行一次搜索，过程如下：

- 从对象实例本身开始。如果找到了该属性，则返回属性的值。
- 如果没有，则继续搜索该对象 `[[prototype]]` 属性所指向的原型对象，如果在原型对象上找到了该属性，则返回属性的值
- 如果没有，则重复上一步，直到原型链末端


那么原型链末端在哪？实际上，所有的引用类型默认都继承自 `Object`，而这个继承也是通过原型链实现的。换句话说：函数默认的原型对象同样会包含一个内部属性 `[[prototype]]`，它指向 `Object.prototype`。这也正是所有自定义类型都汇继承 `toString()`、`valueOf()` 等默认方法的根本原因。

前面例子中展示的原型链少了 `Object` 这一环，补充完整后如下所示：

![](http://ofpfd31sq.bkt.clouddn.com/%E7%90%86%E8%A7%A3%20JavaScript%20%E7%9A%84%E5%8E%9F%E5%9E%8B%E5%AF%B9%E8%B1%A1%E5%92%8C%E5%8E%9F%E5%9E%8B%E9%93%BE-%E5%9B%BE3.svg)



通过上面内容，我们可以看出，原型链实现继承的本质就在于：重写原型对象，代之以一个“父类”的实例。


## 补充

### 原型的动态性
我们对原型对象所做的任何修改都能够立即从实例上反映出来，即使先创建了实例后修改原型也是如此。

### 确定原型和实例之间的关系
有两种方式：

- `instanceof` 操作符：只要用这个操作符来测试实例与原型链中出现过的构造函数，结果就会返回 true
- `isPrototypeOf()` 方法：只要是原型链中出现过的原型，都可以说是该原型链所派生的实例的原型，结果就会返回 true

### 判断属性的位置

`hasOwnProperty()`

### 获取 [[prototype]] 的值

前面提到了，`[[prototype]]` 属性是内部属性，我们没办法直接访问它但是通过 ECMAScript 5 引入的一个方法 `Object.getPrototypeOf()`，它会返回实例对象的 `[[prototype]]` 属性值。例如：

```
Object.getPrototypeOf(person1);
```

参考资料：

- 《JavaScript 高级程序设计》第六章
- http://www.2ality.com/2012/10/javascript-properties.html
