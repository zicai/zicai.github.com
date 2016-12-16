---
layout: post
category : javascript
title: "理解 javascript 中的运算符"
tagline: "Supporting tagline"
tags : []
---

## 逻辑运算符
### 逻辑非
逻辑非操作符用叹号表示 `!`

可以应用于任何数据类型，逻辑非操作符首先会把操作数转换为布尔值，然后再求反。

如果操作数为空字符串、0、null、NaN、undefined，返回 true。

`!!` 构造是一个可以将任意 JavaScript 表达式转化为其等效布尔值得简单方式。
`!!variable` 的结果与 `Boolean(variable)` 相同。

逻辑运算符通常用于布尔型（逻辑）值；这种情况，它们返回一个布尔型值。然而，`&&` 和 `||` 运算符实际上返回一个指定操作数的值，因此这些运算符也用于非布尔型，它们返回一个非布尔型值。

逻辑运算符属于短路操作，如果第一个操作数能决定结果，那么就不会对第二个操作数求值。
### 逻辑与

如果第一个操作数求值结果为 false，就不会对第二个操作数求值了。

```
expr1 && expr2
```

如果 expr1 能转换成 false 则返回 expr1，否则返回 expr2。

### 逻辑或

如果第一个操作数求值结果为 true，就不会对第二个操作数求值了。

```
expr1 || expr2
```
如果 expr1 能转换成 true 则返回 expr1，否则返回 expr2。 



```
var myObject = preferredObject || backupObject
```

## 相等运算符
[https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Comparison_Operators](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Comparison_Operators)

### 相等和不相等
比较之前，会将两个操作数转为相同的类型，再进行全等比较。使用 [Abstract Equality Comparison Algorithm](http://www.ecma-international.org/ecma-262/5.1/#sec-11.9.3)

转换顺序：

1. 有布尔值，先转为数值。false 转换为 0，true 转换为 1
2. 有 number，都转 number
3. 有 string，都转 string
4. 如果有一个操作数为对象，而另一个不是，则调用对象的 `valueOf()` 方法（因为，`valueOf()` 方法用来返回对象的字符串、数值或者布尔值表示），用得到的基本类型值按前面的规则进行比较。

注意：null 和 undefined 是不会做任何转换的。

比较规则：

* null == undefined
* NaN 跟谁都不等，包括自身
* 如果两个操作数都是对象，只有当二者引用的是同一个对象时才相等。

[JavaScript Equality Table](http://dorey.github.io/JavaScript-Equality-Table/)

### 全等和不全等
仅当两个操作数类型相同且值相等时，才为 true。仅比较不转换。使用 [The Strict Equality Comparison Algorithm](http://www.ecma-international.org/ecma-262/5.1/#sec-11.9.6)

## 关系运算符
大于、大于等于、小于、小于等于
 
## 条件(三元)运算符

```
variable = bool_expression ? true_value : false_value;
```

基于对 `bool_expression` 求值的结果，决定给变量 `variable` 赋什么值。如果值为 `true`，则赋值为 `true_value`，否则赋值为 `false_value`。

## 逗号运算符
可以在一条语句中执行多个操作，例如：

```
var num1=1, num2=2, num3=3;
```

## void 运算符
[https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/void](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/void)

void 运算符用来执行给定的表达式，然后返回 undefined。

```
void expression
```

常用来获取 undefined 的原始值，也可以直接使用全局变量 undefined（前提是它没有被赋值为非默认值）

```
void(0)
// 等同于
void 0
```

常见于 javascript 伪协议，浏览器调用 JavaScript 引擎，执行后面代码，并用代码返回结果替换页面内容，除非返回值是 undefined。

```
<a href="javascript:void(0);">
  Click here to do nothing
</a>

<a href="javascript:void(document.body.style.backgroundColor='green');">
  Click here for green background
</a>
```

## typeof 运算符
[https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/typeof](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/typeof)

```
typeof operand
```
The typeof operator returns a string indicating the type of the unevaluated operand.

注意：对于尚未声明过的变量，只能执行一项操作，即使用 typeof 操作符检测其数据类型。

typeof 的返回值汇总如下：

|类型|结果|
|:--:|:--|
|Undefined|	"undefined"|
|Null|	"object" (see below)|
|Boolean|	"boolean"|
|Number|	"number"|
|String|	"string"|
|Symbol (new in ECMAScript 2015)	|"symbol"|
|Host object (provided by the JS environment)|	Implementation-dependent|
|Function object (implements [[Call]] in ECMA-262 terms)|	"function"|
|Any other object|	"object"|

注意下面这些：

```
// Numbers
typeof Math.LN2 === 'number';
typeof Infinity === 'number';
typeof NaN === 'number'; // Despite being "Not-A-Number"

// Symbols
typeof Symbol() === 'symbol'
typeof Symbol('foo') === 'symbol'
typeof Symbol.iterator === 'symbol'

// use Array.isArray or Object.prototype.toString.call
// to differentiate regular objects from arrays
typeof [1, 2, 4] === 'object';

typeof new Date() === 'object';

// Functions
typeof function(){} === 'function';
typeof class C {} === 'function';
typeof Math.sin === 'function';
```

```
// This stands since the beginning of JavaScript
typeof null === 'object';
```
In the first implementation of JavaScript, JavaScript values were represented as a type tag and a value. The type tag for objects was 0. null was represented as the NULL pointer (0x00 in most platforms). Consequently, null had 0 as type tag, hence the bogus typeof return value

## instanceof 运算符
[https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/instanceof](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/instanceof)

instanceof 运算符用来测试 constructor.prototype 是否出现在对象的原型链中。

注意：instanceof 的结果会随着构造函数的 prototype 属性而变化。

### 在多上下文中（例如：frames 或 windows）的 instanceof
不同的 scope 有不同的执行环境。换句话说，它们有不同的内置对象（不同的全局对象，不同的构造函数等等）

### 对字面量使用 instanceof

```
var simpleStr = "This is a simple string"; 
simpleStr instanceof String; // returns false

var newStr    = new String("String created with constructor");
newStr instanceof String; // returns true

var myObj     = {};
myObj instanceof Object;  // returns true, despite an undefined prototype
({})  instanceof Object;    // returns true, same case as above
```

## 运算符优先级

运算符的优先级决定了表达式中运算执行的先后顺序。

### 运算符的结合性
结合性决定了拥有相同优先级的运算符的执行顺序。

例如，赋值运算符结合性为从右到左。且返回结果为运算符右边的那个值。

```
a = b = 5;
// 结果 a 和 b 值都会成为 5
```

参考资料：

- [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_Operators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_Operators)
- [运算符优先级](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Operator_Precedence)

