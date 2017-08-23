---
layout: post
category : javascript
title: "理解 javascript 中的对象"
tagline: "Supporting tagline"
tags : [object]
---

> ECMAScript is object-based: basic language and host facilities are provided by objects, and an ECMAScript program is a cluster of communicating objects. In ECMAScript, an *object* is a collection of zero or more *properties* each with *attributes* that determine how each property can be used

---- [4.2 ECMAScript Overview](https://www.ecma-international.org/ecma-262/8.0/index.html#sec-ecmascript-overview)


### 内容

- 属性类型
- 定义属性
- 检测属性
- 删除属性
- 枚举属性
- 属性特性（property Attribute）
- 防止对象被修改

## 属性（property）类型
属性分为三种：

- 数据属性（data properties）
- 访问器属性（accessor properties）
- 内部属性（Internal properties）：这些属性只在规范中使用，不能通过脚本直接访问。通常表示为[[propertyName]]


## 定义属性

当一个属性第一次添加到一个对象上时，JavaScript 会使用一个内部方法 `[[Put]]`
当一个已有的属性被赋予一个新值时，调用的是 `[[Set]]` 方法

属性名可以是任意合法的 JavaScript 字符串，或者可以被转为字符串的。
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_Objects#Objects_and_properties

### 定义访问器属性

当你想要在给属性赋值的同时触发一些行为或是读取的属性值需要通过计算才能得到的时候，访问器属性会很有用。



方法一：在对象字面量中定义

```javascript
var person = {
    _name: 'TaLuo',
    get name() {
        console.log('reading name');
        return this._name;
    },
    set name(value) {
        console.log('setting name to %s', value);
        this._name = value;
    }
};

console.log(person.name);

person.name = 'Gates';
console.log(person.name);
```

方法二：通过 `defineProperty` 方法

```javascript
var person = {
    _name: 'TaLuo'
};

Object.defineProperty(person, 'name', {
    get: function () {
        console.log('getting name');
        return this._name;
    },
    set: function (value) {
        console.log('setting name');
        this._name = value;
    }
});
```

方法三：通过 `Object.prototype.__defineGetter__` 或 `Object.prototype.__defineSetter__` 方法

```javascript
var person = {
    _name: 'TaLuo'
};

person.__defineGetter__('name', function () {
    console.log('getting name');
    return this._name;
});
person.__defineSetter__('name', function (value) {
    console.log('setting name');
    this._name = value;
});
```

方法三已被 deprecated，优先使用前两种方法。

除了可以给用户定义的对象添加访问器属性，还可以给预定义的核心对象添加访问器属性。



需要注意的几点：

- 如果你仅定义了 getter，该属性就变成只读，在非严格模式下试图写入将失败，而在严格模式下将抛出错误。如果仅定义 setter，该属性就变成只写，在两种模式下试图读取都将失败。
- 方法一中 `[gs]et name(){ }` 的 name 是属性名，而非 getter 和 setter 本身的名字。要想明确的给 getter 或 setter 命名，需要使用后两种方法。[参见](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_Objects#Defining_getters_and_setters)
- Getter 提供另一个好处就是延迟计算，只有访问时才进行计算。甚至可以加上缓存机制，做成 [memoized getters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get#Smart_self-overwriting_lazy_getters)



## 检测属性

由于属性可以随时添加，所以经常需要判断属性是否存在。

不要使用下面方法，因为当 name 值为 falsy(null、undefined、0、false、NaN) 时，会导致错误的判断。

```javascript
if(person.name){
	// do something
}
```

更加可靠的判断属性是否存在的方法是使用 in 操作符。

```javascript
if("name" in person){
	// do something
}
```

in 操作符一个额外的好处是不会评估属性的值。

需要注意的是：in 操作符会检查自有属性和原型属性。所有对象都有一个 `hasOwnProperty()` 方法，当给定属性是自有属性时，返回 true。

## 删除属性
设置一个属性的值为 null 并不会从对象中彻底移除该属性。delete 操作符删除属性。delete 只能移除自有属性。删除访问器属性与删除数据属性一样，都是使用 `delete` 操作符。


```javascript
delete person.name
```

## 枚举属性

有三种种方法：

- for-in 循环

    ```javascript
    for (property in object){
	    console.log(property) // 属性名
	    console.log(object[property]) // 属性值
    }
    ```
- `Object.keys(obj)` 获取自身可枚举属性。
- `Object.getOwnPropertyNames(obj)` 获取自身所有属性（包括不可枚举属性）

三者对比如下：

|方法|是否包含原型属性|是否包含不可枚举属性|
|:---|:---:|:---:|
|for...in|是|否|
|Object.keys|否|否|
|Object.getOwnPropertyNames|否|是|


可枚举属性的 Enumerable 特性都被设置为 true。你添加的属性默认都是可枚举的。而对象的大部分原生方法（Object.prototype）都是不可枚举的。

每个对象都有一个 `propertyIsEnumerable()` 方法，用来判断一个属性是否可枚举。

```javascript
person.propertyIsEnumerable('name')
```



## 属性特性（Property attributes）

- 数据属性和访问器属性通用的属性特性
    - Enumerable
    - Configurable
- 数据属性的属性特性
    - Value
    - Writable
- 访问器属性的属性特性：访问器属性不需要保存值，所以不需要 Value 和 Writable。
    - Get：保存着 getter 函数
    - Set：保存着 setter 函数


相关方法：

- `Object.defineProperty(obj,propertyName,descriptorObj)` 定义或者修改属性特性
- `Object.defineProperties(obj,{propery1:descriptorObj,propery2:descriptorObj})` 同时定义多个属性及其特性
- `Object.getOwnPropertyDescriptor()` 获取属性特性

使用示例：

```javascript
let person  = {}
Object.defineProperty(person, "name", {
    value: "Taluo",
    enumerable: true,
    configurable: true,
    writable: true
})
```

注意：在使用 `Object.defineProperty` 定义新属性时，enumerable、configurable、writable 默认值均为 false。




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

要记住，JavaScript 中的对象是动态的，可以在代码执行的任意时刻发生改变。而基于类的语言会把根据类的定义锁定对象，JavaScript 中的对象没有这种限制。

参考资料：

- 《the principles of object-oriented javascript》chapter 3: understanding objects
- http://www.2ality.com/2012/10/javascript-properties.html