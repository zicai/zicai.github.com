---
layout: post
category : javascript
title: "JavaScript"
tagline: "Supporting tagline"
tags : []
---

## 操作符
### 布尔操作符
#### 逻辑非
逻辑非操作符用叹号表示 `！`

可以应用于任何数据类型，逻辑非操作符首先会把操作数转换为布尔值，然后再求反。

如果操作数为空字符串、0、null、NaN、undefined，返回true。

`!!` 构造是一个可以将任意 JavaScript 表达式转化为其等效布尔值得简单方式。
`!!variable` 的结果与 `Boolean(variable)` 相同。

逻辑运算符通常用于布尔型（逻辑）值；这种情况，它们返回一个布尔型值。然而，&&和||运算符实际上返回一个指定操作数的值，因此这些运算符也用于非布尔型，它们返回一个非布尔型值。

逻辑运算符属于短路操作，如果第一个操作数能决定结果，那么就不会对第二个操作数求值。
#### 逻辑与

如果第一个操作数求值结果为 false，就不会对第二个操作数求值了。

```
expr1 && expr2
```

如果 expr1 能转换成 false 则返回 expr1，否则返回 expr2。

#### 逻辑或

如果第一个操作数求值结果为 true，就不会对第二个操作数求值了。

```
expr1 || expr2
```
如果 expr1 能转换成 true 则返回 expr1，否则返回 expr2。 



```
var myObject = preferredObject || backupObject
```

 
### 条件操作符

```
variable = bool_expression ? true_value : false_value;
```

基于对 `bool_expression` 求值的结果，决定给变量 `variable` 赋什么值。如果值为 `true`，则赋值为 `true_value`，否则赋值为 `false_value`。

### 逗号操作符
可以在一条语句中执行多个操作，例如：

```
var num1=1,num2=2,num3=3;
```

运算符优先级 https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Operator_Precedence

## 流程控制
### if 语句

```
if (i>25){
	console.log('25')
}else if(i>30){
	console.log('30')
}else{
	console.log('40')
}
```

ECMAScript 会自动调用Boolean()转换函数将条件表达式的结果转换为布尔值

### do-while 语句

```
do{
	console.log('do')
}while(expression)
```

### while 语

```
while(expression)
	statement
```
	
### for 语句

```
for(initialization;expression;post-loop-expression)
	statement
```

由于ECMAScript中不存在块级作用域，所以在循环内部定义的变量在外部也可以访问到。

for 语句中的初始化表达式，控制表达式和循环后表达式都是可选的。

### for-in 语句

for-in 语句可以用来枚举对象的属性。

```
for (property in expression)
	statement
	
```
ECMAScript 对象的属性是没有顺序的，所以for-in语句循环的属性的顺序是不可预测的。

建议在使用for-in之前，先检查对象的值是否是null或是undefined。

### break 和 continue 语句

break语句会立即退出循环，强制继续执行循环后面的语句。
continue语句也是立即退出循环，但是退出之后会从循环的顶部继续执行。

### switch 语句

```
switch（expression）{
	case value:
		statement;
		break;
	case value:
		statement;
		break;
	default:
		statement;
}
```

每一个case的含义是，如果表达式的值等于这个值value，则执行后面的statement。break会跳出switch语句。表达式不匹配任何一种情形时会执行default语句。

可以在switch语句中使用任何数据类型。每个case 的值不一定是常量，可以是变量，也可以是表达式。

switch语句在比较时使用的是全等操作符，不会发生类型转换。



## 数据类型

五种简单类型：

- undefined
- null
- boolean
- number
- string

一种复杂类型：

- Object

Object 有几种子类型：

- Function
- Array
- Date
- RegExp
- Error
- String
- Number
- Boolean

### 数据类型检测
简单类型除了 null 之外，都可以用 typeof 操作符。返回值分别为："string","number","boolean","undefined"

注意：对未声明的变量和未初始化的变量使用 typeof 都会返回 "undefined"

如果期望的值为 null ,使用 === 或 !=== 来和 null 比较。

检测引用值使用 instanceof

检测数组可以用 `Array.isArray()`

### undefined
没有被初始化的变量都有一个初始值，即 undefined，表示该变量等待被赋值。包括未赋值的函数参数。

注意：

- null == undefined 为 true


### null
把 null 理解为对象的占位符。

### Number 类型
NaN 是一个特殊的数值，它有两个特点：

- NaN 与任何值都不相等，包括自身
- 任何涉及 NaN 的操作都返回 NaN

isNaN() 函数接受一个参数，任何不能转换为数值的值都会返回 true。注意：boolean 值可以转换为 0 或 1。

### String 类型

String 类型包含一些特殊的字符字面量，也叫转义序列，例如：

- `\t`
- `\n`
- `\b`
- `\\`
- `\'`
- `\"`
- `\xnn`
- `\unnnn`

常用方法：

- 按长度切割：
	
		str.substr(start[, length])
		
- 按位置切割：

		str.slice(beginSlice[, endSlice])
		str.substring(indexStart[, indexEnd])

- 查找子字符串：

		str.indexOf(searchValue[, fromIndex])
		str.lastIndexOf(searchValue[, fromIndex])
	
	如果没找到返回 -1

- 按正则匹配：
	
	- 只想知道是否匹配：`regexObj.test(str)`
	- 想知道是否匹配，同时要获取位置：`str.search(regexp)` 只接收一个参数，正则表达式或者 RegExp 对象。返回字符串中第一个匹配项的索引，如果没有找到，则返回-1。
	- 更多信息：`str.match(regexp)` 本质上与调用RegExp 的 `exec()` 方法相同。只接收一个参数，正则表达式或者 RegExp 对象。返回结果是一个数组：数组的第一项是与整个模式匹配的字符串，之后的每一项（如果有）保存着与正则表达式中的捕获组匹配的字符串

- `replace()`：接收两个参数：第一个参数是一个 RegExp 对象或者一个字符串（这个字符串不会被转换成正则表达式），第二个参数是一个字符串或者函数。如果第一个参数是字符串，那么只会替换第一个子字符串。要想替换所有子字符串，唯一的办法就是提供一个正则表达式，而且要指定全局标志。
- `split()`：基于指定的分隔符将一个字符串分割为多个字符串，并将结果放到一个数组中。分割符可以使字符串，也可以是 RegExp 对象。它还接收可选的第二个参数，用于指定数组的大小，以确保返回的数组不会超过既定大小。`str.split('')` 就可以将字符串转为数组



### Array 类型

常用方法：

- concat：合并多个数组 `var new_array = old_array.concat(value1[, value2[, ...[, valueN]]])`
- reverse：逆序
- sort：排序，默认是根据 string Unicode code points 排序
- join：数组转为字符串，默认是逗号分隔。`arr.join('')` 拼成字符串。
- keys：返回数组迭代器
- values：返回数组迭代器

查找元素：

- includes：判断是否包含某元素
- indexOf：返回给定元素第一次出现的索引值
- lastIndexOf：返回给定元素最后一次出现的索引值

操作元素：

- pop：移除最后一个元素，并返回该元素
- push：在数组尾部添加一个或多个元素，返回数组长度
- shift：移除第一个元素，并返回该元素
- unshift：在数组头部添加一个或多个元素，返回数组长度
- slice：切割，返回新数组 `arr.slice(begin, end)`
- splice：剪接当前数组，删除或者新加项

循环：

- every：是否所有元素都满足回调
- some：是否有元素满足回调
- forEach：`arr.forEach(callback[, thisArg])` callback 参数依次为 currentValue、index、array
- map：返回新数组
- filter：返回满足回调函数的项组成的新数组
- findIndex：返回第一个满足回调的项的索引值
- find：返回第一满足回调的项
- reduce：合并为一个值
- reduceRight：合并为一个值

去重

```
function unique(arr) {
  var n = []; //一个新的临时数组
  for (var i = 0; i < arr.length; i++) {
    //如果当前数组的第i已经保存进了临时数组，那么跳过，
    //否则把当前项push到临时数组里面
    if (n.indexOf(arr[i]) == -1) {
      n.push(arr[i]);
    }
  }
  return n;
}
```

### Object 类型

 检测属性
使用in运算符

```
if("count" in object){
	a=object.count
}
```


### Function 类型

### Date 类型

新建日期对象时必须按照构造函数方式使用 `new` ,如果以常规方法方式调用 `Date()` ，那么只会返回字符串，而不是日期对象。和其它 JavaScript 对象不同，日期对象没有 literal syntax。
	
- 获取毫秒数
	
		Date.now() // 返回当前时间的毫秒数。
		dateObj.valueOf() // 返回特定时间的毫秒数。

注意：时间戳是秒数。
### RegExp 类型
- 新建正则

		var expression=/pattern/flags
		var pattern=new RegExp("[bc]at","i") // 传给RegExp构造函数的两个参数都是字符串（不能把正则表达式字面量传递给构造函数）

	模式中使用的所有元字符都需要转义。元字符包括：


		（[{\^$|)?*+.]}


	由于RegExp构造函数的模式参数是字符串，所以在某些情况下要对字符进行双重转义。所有元字符都必须双重转义，那些已经转移过的字符也是如此。

### 单体内置对象
内置对象是指：由 ECMAScript 实现提供、不依赖于宿主环境，这些对象在 ECMAScript 程序执行前就已经存在了。换句话说，开发人员不必显式的实例化内置对象，因为它们已经实例化了。除了前面介绍的 Object、Array、String 等内置对象，ECMAScript-262 还提供了两个单体内置对象：Global 和 Math 

#### Global

事实上，没有全局变量或全局函数，所有在全局作用域中定义的属性和函数，都是`Global`对象的属性。

它包含的方法有：

- `isNaN()`
- `isFinite`
- `parseInt()`
- `parseFloat()`
- `encodeURIComponent()`
- `decodeURIComponent()`
- `eval()`


#### Math 对象

- `Math.ceil()` 向上舍入
- `Math.floor()` 向下舍入
- `Math.round()` 标准舍入
- `random()` 方法返回一个介于0到1之间的一个随机数，不包括0和1。

从某个整数范围内随机选择一个值

```
值 = Math.floor( Math.random() * 可能值的总数 + 第一个可能的值)

// 返回指定范围的随机数，包含最小值 min，不包含最大值 max
function getRandomArbitrary(min, max) {
  return Math.random() * (max - min) + min;
}

// 返回指定范围的随机整数，包含最小值 min，不包含最大值 max
// Using Math.round() will give you a non-uniform distribution!
function getRandomInt(min, max) {
  return Math.floor(Math.random() * (max - min)) + min;
}
```


## 继承

### 原型链实现继承

```
function SuperType() {
  this.name = 'super type'
}

SuperType.prototype.getName = function() {
  return this.name
}

function SubType() {
  this.year = '1';
}

SubType.prototype = new SuperType();

SubType.prototype.getYear = function() {
  return this.year;
}

var instance = new SubType();
console.log(instance.getName());

```
原型链实现继承的问题在于：SuperType 的实例属性成了 SubType 的原型属性。换句话说：多个 SubType 实例会共享 SuperType 的实例属性（引用类型）

### 借用构造函数模式（也叫伪造对象或经典继承）
思路是：在子类型构造函数内部调用超类型构造函数。

```
function SuperType() {
	this.colors=['red','yellow']
}

function SubType() {
	SuperType.call(this);
}

```

### 组合继承（也叫伪经典继承）
将上面两种技术组合。思路是：原型链实现对原型属性和方法的继承，而通过构造函数来实现对实例属性的继承。

```
function SuperType(name) {
  this.name = name;
  this.color = ['red', 'yellow']
}

SuperType.prototype.sayName = function() {
  alert(this.name)
}

function SubType(name, age) {
  SuperType.call(this, name);
  this.age = age;
}

SubType.prototype = new SuperType();

SubType.prototype.sayAge = function() {
  alert(this.age)
}

var instance1 = new SubType('one', 1);
instance1.sayAge()

```

### 原型式继承
没有使用严格意义上的构造函数。直接基于已有对象创建新对象。


ECMAScript 5 通过新增 Object.create() 方法规范化了原型式继承。

```
var person = {
	name:'lyg',
	friends:['hm','hh']
}

var anotherPerson = Object.create(person);

```

## 模块

```
var MODULE = (function() {
  var my = {},
    privateVariable = 1;

  function privateMethod() {
    // ...
  }

  my.moduleProperty = 1;
  my.moduleMethod = function() {
    // ...
  };

  return my;
}());
```

## this

bind 简单实现

```
function bind(fn,obj){
	return function(){
		return fn.apply(obj,arguments)
	}
}
```



```
fun.apply(thisArg, [argsArray])
fun.call(thisArg[, arg1[, arg2[, ...]]])
```

## 轮播图

```
var index = 0;
var len = 3;
var sWidth=100;

function showPics(index) {
  var nowLeft = -index * sWidth; //根据index值计算ul元素的left值
  $(".container ul").stop(true, false).animate({
    "left": nowLeft
  }, 300); //通过animate()调整ul元素滚动到计算出的position
}

setInterval(function() {
  showPics(index);
  index++;
  if (index == len) {
    index = 0;
  } //设置循环，当前index如果等于图片数目，index置0
}, 4000); //此4000代表自动播放的间隔，单位：毫秒
```

## deepequal

```
var deepEqual = function(x, y) {
  if ((typeof x == "object" && x != null) && (typeof y == "object" && y != null)) {
    if (Object.keys(x).length != Object.keys(y).length)
      return false;

    for (var prop in x) {
      if (y.hasOwnProperty(prop)) {
        if (!deepEqual(x[prop], y[prop]))
          return false;
      } else
        return false;
    }

    return true;
  } else if (x !== y)
    return false;
  else
    return true;
}
```

## 发布订阅

```
(function($) {

  var o = $({});

  $.subscribe = function() {
    o.on.apply(o, arguments);
  };

  $.unsubscribe = function() {
    o.off.apply(o, arguments);
  };

  $.publish = function() {
    o.trigger.apply(o, arguments);
  };

}(jQuery));
```

