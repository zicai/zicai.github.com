---
layout: post
category : javascript
title: "理解 javascript 中的对象"
tagline: "Supporting tagline"
tags : [object]
---

> ECMAScript is object-based: basic language and host facilities are provided by objects, and an ECMAScript program is a cluster of communicating objects. In ECMAScript, an *object* is a collection of zero or more *properties* each with *attributes* that determine how each property can be used

---- [4.2 ECMAScript Overview](https://www.ecma-international.org/ecma-262/8.0/index.html#sec-ecmascript-overview)

本文分为三部分：

- 第一部分：对象
- 第二部分：属性
- 第三部分：禁止修改对象

## 第一部分：对象

内容：

- 对象定义
- 对象类型
- 对象复制

### 对象定义
两种方式：

- 对象字面量
- 构造函数

### 对象类型

内置对象类型

* String
* Number
* Boolean
* Object
* Function
* Array
* Date
* RegExp
* Error
* 

它们实际上是一些内置函数，可以用作构造函数。

对 Object，Array，Function，RegExp 来说，无论使用字面量创建还是构造函数创建，它们都是对象。

Date 只有构造函数形式，没有字面量形式

String、Number 只有以构造函数形式创建出来的才是对象，否则就是基本类型

null 和 undefined 只有字面量，没有对应的构造函数


### 对象复制

* 浅复制：ES6 引入的 `Object.assign()`
* 深复制：要考虑循环引用的问题
    * 对于JSON安全的对象，可以使用 `var newObj = JSON.parse( JSON.stringfy( somObj ) )`

## 第二部分：属性


- 属性类型
- 定义属性
- 访问属性
- 检测属性
- 删除属性
- 枚举属性
- 属性特性（property Attribute）
- 防止对象被修改

### 属性（property）类型
属性分为三种：

- 数据属性（data properties）
- 访问器属性（accessor properties）
- 内部属性（Internal properties）：这些属性只在规范中使用，不能通过脚本直接访问。通常表示为[[propertyName]]


### 定义属性

当一个属性第一次添加到一个对象上时，JavaScript 会使用一个内部方法 `[[Put]]`
当一个已有的属性被赋予一个新值时，调用的是 `[[Set]]` 方法

属性名可以是任意合法的 JavaScript 字符串，或者可以被转为字符串的。
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_Objects#Objects_and_properties

#### 定义访问器属性

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


### 访问属性

* . 操作符要求属性名满足标识符的命名规范
* [‘'] 则可以接受任意的 UTF8 字符串作为属性名，也可以是变量
ES6 增加了可计算属性名，最常用场景可能是 ES6 的 Symbol。

### 检测属性

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

需要注意的是：in 操作符会检查所有属性，包括自有属性和原型属性（不论是否可枚举）。所有对象都有一个 `hasOwnProperty()` 方法，当给定属性是自有属性时，返回 true。

### 删除属性
设置一个属性的值为 null 并不会从对象中彻底移除该属性。delete 操作符删除属性。delete 只能移除自有属性。删除访问器属性与删除数据属性一样，都是使用 `delete` 操作符。


```javascript
delete person.name
```

### 枚举属性

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

注意：在数组上应用 for...in 有时会产生出人意料的结果，因为这种枚举不仅会包含所有数值索引，还会包含所有可枚举属性。最好只在对象上应用 for..in 循环，如果要遍历数组就使用传统的 for 循环

### 属性特性（Property attributes）

属性特性或者称为属性描述符。

在 ES5 之前，JS 语言本身没有提供可以直接检测属性特性的方法，比如判断属性是否可读。但是从 ES5 开始，所有属性都有了属性描述符

- 数据属性和访问器属性通用的属性特性
    - Enumerable
    - Configurable：如果为 false，无法删除对应属性，无法修改对应属性的特性（例外：可以把 writable 由 true 改为 false）
- 数据属性的属性特性
    - Value
    - Writable：如果为 false，修改对应属性值会静默失败（严格模式下会抛出 TypeError）
- 访问器属性的属性特性：访问器属性不需要保存值，所以不需要 Value 和 Writable。
    - Get：保存着 getter 函数
    - Set：保存着 setter 函数


相关方法：

- `Object.defineProperty(obj,propertyName,descriptorObj)` 给对象添加一个新属性或修改一个已有属性，同时对属性特性进行修改。
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

结合 writable 和 configurable 可以给对象创建常量属性（不可修改、重定义、删除）

## 第三部分：禁止修改对象

JavaScript 中的对象是动态的，可以在代码执行的任意时刻发生改变。而基于类的语言会把根据类的定义锁定对象，JavaScript 中的对象没有这种限制。不过有的时候，也需要防止特定 JavaScript 对象被修改。

对象和属性一样具有指导其行为的内部特征。其中，[[Extensible]] 是一个布尔值，用来指明对象本身是否可以被修改。

防止对象被修改的三种方式：

- preventExtensions(禁止扩展)
- seal(密封)
- freeze(冻结)

### preventExtensions

经过 `Object.preventExtensions(obj)` 之后，就不能给 obj 添加新的属性

`Object.isExtensible(obj)` 用来检查 [[Extensible]] 的值

### seal

经过 `Object.seal(obj)` 之后，就不能给 obj 添加新属性，也不能改变其类型（从数据属性变成访问器属性，或者相反）或者删除任何现有属性（但是可以修改属性值）。

`Object.seal(obj)` 实际上是调用了 `Object.preventExtensions(obj)`，同时把所有现有属性标记为 `configurable:false`

`Object.isSealed(obj)` 用来判断一个对象是否被密封。

在 Java 或者 C++ 中，基于类实例化的对象，你无法给对象添加新的属性，但是可以修改属性的值。JavaScript 中的密封可以达到同样的效果。

### freeze
`Object.freeze(obj)` 是级别最高的不可变性，禁止对对象本身及其任意直接属性的修改。

实际上是调用了 `Object.seal(obj)`，同时把所有“数据访问”属性标记为 `writable:false`

`Object.isFrozen(obj)` 用来判断一个对象是否被冻结。

注意：

- 所有的不变性都是浅不变性，也就是说，它们只会影响目标对象和它的直接属性
- 应该在严格模式下，使用上述防止对象被修改的方法。因为在非严格模式下，违反规则通常只会静默失败，而不会抛出异常。



参考资料：

- 《你不知道的 JS》-- Object
- 《the principles of object-oriented javascript》chapter 3: understanding objects (中译名：JavaScript 面向对象精要)
- http://www.2ality.com/2012/10/javascript-properties.html