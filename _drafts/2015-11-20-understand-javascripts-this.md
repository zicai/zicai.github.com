---
layout: post
category : lessons
title: "理解 JavaScript 的 this"
tagline: "Supporting tagline"
tags : [javascript]
---

## 第一部分：函数与 this

在 Java、PHP 这样的语言中，类方法中的 `this` 被看做当前对象的实例。它不能在方法之外使用，简单没有歧义。

在 JavaScript 中，`this` 是一个函数当前的执行上下文。由于 JS 有四种调用函数的方式：

- 函数调用：`alert('Hello World!')`
- 方法调用：`console.log('Hello World!')`
- 构造函数调用：`new RegExp('\\d')`
- 非直接调用：`alert.call(undefined, 'Hello World')`

每一种调用方式都对应不同的上下文。

而且 strict mode 也会影响执行上下文。

理解 `this` 的关键在于要对函数调用如何影响上下文有一个清晰地认识。

第一部分专注于解释函数调用、函数调用如何影响 `this` 以及确定上下文时常见的误区。

开始之前，先来熟悉一些词汇：

- Invocation
- Context
- Scope


## 函数调用
### 函数调用中的 this 值

> 在函数调用中 this 值是全局对象

全局对象由执行环境决定。在浏览器中是 window 对象。

在全局执行上下文中（global execution context），`this` 引用的也是全局对象。

### 严格模式下，函数调用中的 this 值


> 严格模式下，函数调用中的 this 值为 undefined

ECMAScript 5.1 引入了严格模式。
 
严格模式不仅会影响当前 scope，同时会影响 inner scope（所有内部声明的函数）。
 
 
### 误区：内部函数的 this 值（shadowing this）
 
一个常见的陷阱是：认为内部函数的 this 值与外部函数一样。实际上，内部函数的上下文只取决于调用方式，而非外部函数的上下文。

在内部函数中，你是不能用 `this` 引用到外部函数的 `this` 值的，因为外部函数 `this` 值被 shadowed. 为了获得外部函数的 `this` 值，通常需要使用 `.call()`，`.apply()` 或 `.bind()`。例如：

```
var obj = {
  name: 'Jane',
  friends: ['Tarzan', 'Cheeta'],
  loop: function() {
    'use strict';
    this.friends.forEach(
      function(friend) {
        console.log(this.name + ' knows ' + friend);
      }
    );
  }
};
obj.loop();
// TypeError: Cannot read property 'name' of undefined
```

常用的解决方法有三种：

1. `that = this`

```
loop: function() {
  'use strict';
  var that = this;
  this.friends.forEach(function(friend) {
    console.log(that.name + ' knows ' + friend);
  });
}
```

2. 使用 `bind()`

```
loop: function() {
  'use strict';
  this.friends.forEach(function(friend) {
    console.log(this.name + ' knows ' + friend);
  }.bind(this));
}
```

3. 使用 `forEach` 的第二个参数，它会传递给回调函数作为 `this` 值

```
loop: function() {
  'use strict';
  this.friends.forEach(function(friend) {
    console.log(this.name + ' knows ' + friend);
  }, this);
}
```

除了 `forEach` 之外，接受参数作为 `this` 值的函数还有：

- `Array.prototype.every( callbackfn [ , thisArg ] )`
- `Array.prototype.some( callbackfn [ , thisArg ] )`
- `Array.prototype.map( callbackfn [ , thisArg ] )`
- `Array.prototype.filter( callbackfn [ , thisArg ] )`

## 方法调用

### 方法调用中的 this 值

> 方法调用中 this 值为包含该方法的对象

JavaScript 对象可以从 prototype 中继承方法。当调用这些继承的方法时，调用的上下文仍然是对象本身。

在 ECMAScript 6 `class` 语法中，方法调用的上下文也是实例本身。

### 误区：从对象上分离方法

可以把对象的方法赋值给一个独立的变量，当通过这个变量调用该方法时，你可能会认为 `this` 值为定义方法的对象。事实上，如果方法没有通过对象调用，就成了函数调用。那么，`this` 值就是全局对象 window 或是严格模式下的 undefined。

常见的例子包括 `setTimeout()` 和注册事件处理器（详见第三部分）。

```
/** Similar to setTimeout() and setImmediate() */
function callIt(func) {
  func();
}
```

一个具体的例子：

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

在上面的例子中，当方法作为参数传递时，就从对象上剥离了。使用 `bind()` 可以修正该行为：

```
var myCat = new Animal('Cat', 4);  
// logs "The Cat has 4 legs"
setTimeout(myCat.logInfo.bind(myCat), 1000);  
```

## 构造函数调用

如果构造函数调用时没有参数，那么括号可以省略。

从 ECMAScript 6 开始，可以使用 `class` 关键字定义构造器。

### 构造函数调用中的 this

> 在构造函数调用中，this 的值为新创建的对象。

构造函数调用的上下文是新创建的对象。

### 误区：忘记使用 new
忘记使用 new 就成了函数调用，会出现意料之外的状况。

为了避免忘记使用 `new` 可以使用下面代码

```
function Vehicle(type, wheelsCount) {  
  if (!(this instanceof Vehicle)) {
    throw Error('Error: Incorrect invocation');
  }
  this.type = type;
  this.wheelsCount = wheelsCount;
  return this;
}

// Function invocation. Generates an error.
var brokenCat = Vehicle('Broken Car', 3);  
```

## 非直接调用

非直接调用是指：通过 `.call()` 和 `.apply()` 方法调用函数。

### 非直接调用中的 this 值

> 在非直接调用中 this 值是 .call() 或 .apply() 方法的第一个参数

当需要在特定上下文执行某个函数时，非直接调用是很有用的。例如：可以用来解决函数调用时，this 总是全局对象或严格模式下的 undefined。也可以用来模拟对象上的方法调用。

还有一种常见用法是：在 ES5 中，在子类中调用父类的构造函数。

```
function Runner(name) {  
  console.log(this instanceof Rabbit); // => true
  this.name = name;  
}
function Rabbit(name, countLegs) {  
  console.log(this instanceof Rabbit); // => true
  // Indirect invocation. Call parent constructor.
  Runner.call(this, name);
  this.countLegs = countLegs;
}
var myRabbit = new Rabbit('White Rabbit', 4);  
myRabbit; // { name: 'White Rabbit', countLegs: 4 }  
```


## Bound function

bound function 是指跟一个对象绑定的函数。通常在原始函数上使用 `bind()` 方法来创建。原始函数与 bound function 共享相同的代码和 scope，但是执行上下文不同。

### bound function 中的 this 值

> 在 bound function 中 this 的值为 .bind() 的第一个参数。

`.bind()` 方法用来创建一个新函数，新函数调用时的上下文是传给 `.bind()` 的第一个参数。这个强大的技巧，可以用来创建一个有预定义 this 值的函数。

`.bind()` 创建一个永久的上下文，新函数与其进行关联并保持。在 bound function 上使用 `.call()` 或 `.apply()` 并不会改变关联的上下文，rebound 也不会有任何效果。

只有构造函数调用会改变 bound function 的上下文，但是不推荐这么做。


## 箭头函数

### 箭头函数中的 this

### 误区：

## 结语
因为函数的调用方式直接影响 `this` 的值，所以从现在开始，不要再问：

> Where is this taken from?

而要问：

> How is the function invoked?

对于箭头函数，要问：

> What is this where the arrow function is defined?

PS: 第一部分的内容主要来自 [Gentle explanation of 'this' keyword in JavaScript](http://rainsoft.io/gentle-explanation-of-this-in-javascript)。涵盖了大多数情况下 this 的值（主要是函数里的 this）。第二部分会做一个补充。

## 第二部分：补充

The ECMAScript Standard defines this as a keyword that "evaluates to the value of the ThisBinding of the current execution context" (§11.1.1). ThisBinding is something that the JavaScript interpreter maintains as it evaluates JavaScript code, like a special CPU register which holds a reference to an object. The interpreter updates the ThisBinding whenever establishing an execution context in one of only three different cases:

- Outside functions (in the top-level scope)
- In functions
- In a string passed to eval()

## global context
global execution context（outside of any function）,`this` 引用的是全局对象。无论是否是严格模式。

在 Node.js 中，代码通常都是运行在模块中。所以，top-level scope 是一个特殊的 module scope

```
// `this` doesn’t refer to the global object:
console.log(this !== global); // true
// `this` refers to a module’s exports:
console.log(this === module.exports); // true
```
## function context
在函数里，`this` 的值取决于函数如何调用。如上第一部分所述。

### 箭头函数
arrow functions – functions without their own this. Inside such functions, you can freely use this, because there is no shadowing

### 对象的方法

`this` binding is only affected by the most immediate member reference. 如下所示：

```
var o = {prop: 37};

function independent() {
  return this.prop;
}

o.b = {g: independent, prop: 42};
console.log(o.b.g()); // logs 42
```

A function used as getter or setter has its this bound to the object from which the property is being set or gotten.

```
function sum(){
  return this.a + this.b + this.c;
}

var o = {
  a: 1,
  b: 2,
  c: 3,
  get average(){
    return (this.a + this.b + this.c) / 3;
  }
};

Object.defineProperty(o, 'sum', {
    get: sum, enumerable:true, configurable:true});

console.log(o.average, o.sum); // logs 2, 6
```

## eval
`eval()` 有两种调用方式：详见[这里](http://speakingjs.com/es5/ch23.html#_indirect_eval_evaluates_in_global_scope)

- 直接调用
- [间接调用](http://dmitrysoshnikov.com/ecmascript/es5-chapter-2-strict-mode/#indirect-eval-call)

如果间接调用 `eval()`，`this` 引用的是全局对象。如下：

```
> (0,eval)('this === window')
true
```

如果是直接调用 `eval()`，`this` 值与 `eval()` 外层 `this` 值相同。如下：

```
// Real functions
function sloppyFunc() {
  console.log(eval('this') === window); // true
}
sloppyFunc();

function strictFunc() {
  'use strict';
  console.log(eval('this') === undefined); // true
}
strictFunc();

// Constructors
var savedThis;

function Constr() {
  savedThis = eval('this');
}
var inst = new Constr();
console.log(savedThis === inst); // true

// Methods
var obj = {
  method: function() {
    console.log(eval('this') === obj); // true
  }
}
obj.method();
```

思考题：

```
<script type="text/javascript">
var obj = {
    myMethod : function () {
        // What is the value of this at this line
    }
};
var myFun = obj.myMethod;
myFun();
</script>
```

答案是：window。
This one was tricky. When evaluating the eval code, this is obj. However, in the eval code, myFun is not called on an object, so ThisBinding is set to window for the call.

## 第三部分：实战--事件处理中的 this
参考资料：[http://www.quirksmode.org/js/this.html](http://www.quirksmode.org/js/this.html)

先定义一个函数：

```
function doSomething() {
   this.style.color = '#cc0000';
}
```

### DOM event handler

```
element.onclick = doSomething;
```

`doSomething` 函数会完整的拷贝到 `onclick` 属性上，成为一个方法。

```
alert(element.onclick)
// 会得到
function doSomething()
{
	this.style.color = '#cc0000';
}
```

所以 `this` 值就是绑定事件处理器的元素。

### in-line event handler

```
<element onclick="doSomething()">
```
在这种情况下，不会拷贝函数，只是调用它。

```
alert(element.onclick)
// 会得到
function onclick()
{
	doSomething()
}
```
所以 `this` 值为全局对象。可以改写为：

```
<element onclick="doSomething(this)">

function doSomething(obj) {
	// this is present in the event handler and is sent to the function
	// obj now refers to the HTML element, so we can do
	obj.style.color = '#cc0000';
}
```


参考资料：

- https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this
- http://www.2ality.com/2014/05/this.html
- http://stackoverflow.com/questions/3127429/how-does-the-this-keyword-work