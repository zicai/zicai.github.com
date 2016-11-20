---
layout: post
category : javascript
title: "理解 javascript 中的对象"
tagline: "Supporting tagline"
tags : [object]
---

### 内容

- 定义属性
- 检测属性
- 删除属性
- 枚举属性
- 属性类型
- 属性特性（property Attribute）
- 防止对象被修改

要记住，JavaScript 中的对象是动态的，可以在代码执行的任意时刻发生改变。而基于类的语言会把根据类的定义锁定对象，JavaScript 中的对象没有这种限制。

## 定义属性

当一个属性第一次添加到一个对象上时，JavaScript 会使用一个内部方法 [[Put]]
当一个已有的属性被赋予一个新值时，调用的是 [[Set]] 方法

## 检测属性

由于属性可以随时添加，所以经常需要判断属性是否存在。

不要使用下面方法，因为当 name 值为 falsy(null、undefined、0、false、NaN) 时，会导致错误的判断。

```
if(person.name){
	// do something
}
```

更加可靠的判断属性是否存在的方法是使用 in 操作符。

```
if("name" in person){
	// do something
}
```

in 操作符一个额外的好处是不会评估属性的值。

需要注意的是：in 操作符会检查自有属性和原型属性。所有对象都有一个 hasOwnProperty() 方法，当给定属性是自有属性时，返回 true。

## 删除属性
设置一个属性的值为 null 并不会从对象中彻底移除该属性。
delete 操作符删除属性。delete 只能移除自有属性。

```
delete person.name
```

## 枚举属性

有两种方法：

- for-in 循环

	```
for (property in object){
	console.log(property) // 属性名
	console.log(object[property]) // 属性值
}
```
- `Object.keys(obj)` 可以用来获取可枚举属性的名字的数组。

二者的区别在于 for-in 循环也会遍历原型属性，而 Object.keys() 只返回自有属性。

可枚举属性的 [[Enumerable]] 都被设置为 true。你添加的属性默认都是可枚举的。而对象的大部分原生方法都是不可枚举的。

每个对象都有一个 propertyIsEnumerable() 方法，用来判断一个属性是否可枚举。

```
person.propertyIsEnumerable('name')
```

## 属性类型
属性分为两种：data properties and accessor properties.
Accessor properties are most useful when you want the assignment of a value to trigger some sort of behavior, or when reading a value requires the calculation of the desired return value.

## 属性特性
ECMAScript defines multiple internal properties for objects in JavaScript, and these internal properties are indicated by double-square-bracket notation

    - [[Enumerable]]
    - [[Configurable]]
    - [[Value]]
    - [[Writable]]
    - [[Get]] and [[Set]]

Object .defineProperty() 修改属性特性
Object.defineProperties()
Object.getOwnPropertyDescriptor()

## 防止对象被修改
三种方式：

- Preventing Extensions
    - 对象和属性一样，也具有特性。
[[Extensible]]
Object.preventExtensions()
- Sealing Objects
    - A sealed object is nonextensible, and all of its properties are nonconfigu-rable. That means not only can you not add new properties to the object, but you also can’t remove properties or change their type (from data to accessor or vice versa). If an object is sealed, you can only read from and write to its properties
    - Object.seal()
    - Ineffect, sealed objects are JavaScript’s way ofgiving you the same mea-sure of control without using classes.
- Freezing Objects
    - a frozen object is a sealed object where data properties are also read-only. Frozen objects can’t become  unfrozen
    - Object.freeze()
    - Object.isFrozen()

参考资料：

- 《the principles of object-oriented javascript》chapter 3: understanding objects
- http://www.2ality.com/2012/10/javascript-properties.html