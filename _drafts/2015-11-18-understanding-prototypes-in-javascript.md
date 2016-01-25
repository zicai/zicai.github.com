---
layout: post
category : lessons
title: "理解 JavaScript 的 prototype"
tagline: "Supporting tagline"
tags : [javascript]
---


## a whole new object
在 JavaScript 中，对象就是键值对，创建对象可以使用 `Object.create`:

```
var person = new Object(null); // 创建了一个空对象
```

为什么不使用 `var person = {};` ? 请继续往下看。

## 添加属性

添加属性之前，需要先理解什么是属性，（规范中称为：named data property）。

显然，属性有名字和值。而且，属性可以是 enumerable，configurable，writable。如果某个属性是 enumerable，那么使用 `for(prop in obj)` 迭代对象时会出现该属性。如果是 writable，就可以替换该属性。如果是 configurable ,就可以删除或是改变该属性的 attributes 。

可以通过 `Object.defineProperty` 给对象添加属性。我们给上面的空对象添加 first name 和 last name :

```
var person = Object.create(null);
Object.defineProperty(person, 'firstName', {
	value : "yehuda",
	writable: true,
  	enumerable: true,
  	configurable: true
});

Object.defineProperty(person, 'lastName', {  
  value: "Katz",
  writable: true,
  enumerable: true,
  configurable: true
});
```

很明显，这太啰嗦了。可以稍微简化下：

```
var config = {  
  writable: true,
  enumerable: true,
  configurable: true
};

var defineProperty = function(obj, name, value) {  
  config.value = value;
  Object.defineProperty(obj, name, config);
}

var person = Object.create(null);  
defineProperty(person, 'firstName', "Yehuda");  
defineProperty(person, 'lastName',   "Katz");  
```

当然，上面的代码还是很丑陋。在提出更好的解决方案之前，需要先给 JavaScript 武器库中增加一件武器。

## prototype

上面提到过，对象就是键值对。实际上，JavaScript 对象还有额外的属性：a pointer to another object. We call this pointer the object's prototype. 如果在某个对象上读取属性时未找到，则会沿着原型链向上查找，直到发现 `null`，在这种情况下，返回 `undefined`。

实际上，`Object.create(null)`，参数就是用来告诉 JavaScript 把谁设为对象的原型。可以用 `Object.getPrototypeOf` 来查看对象的原型。

```
var man = Object.create(null);  
defineProperty(man, 'sex', "male");

var yehuda = Object.create(man);  
defineProperty(yehuda, 'firstName', "Yehuda");  
defineProperty(yehuda, 'lastName', "Katz");

yehuda.sex       // "male"  
yehuda.firstName // "Yehuda"  
yehuda.lastName  // "Katz"

Object.getPrototypeOf(yehuda) // returns the man object  
```

也可以添加函数，在多个对象间共享：

```
var person = Object.create(null);  
defineProperty(person, 'fullName', function() {  
  return this.firstName + ' ' + this.lastName;
});

// this time, let's make man's prototype person, so all
// men share the fullName function
var man = Object.create(person);  
defineProperty(man, 'sex', "male");

var yehuda = Object.create(man);  
defineProperty(yehuda, 'firstName', "Yehuda");  
defineProperty(yehuda, 'lastName', "Katz");

yehuda.sex        // "male"  
yehuda.fullName() // "Yehuda Katz"  
```

## 设置属性

由于创建一个 writable, configurable, enumerable 的新属性是很常见的需求，JavaScript提供了赋值语法来简化这一过程。我们用赋值语法替换上面的 `defineProperty`

```
var person = Object.create(null);

// instead of using defineProperty and specifying writable,
// configurable, and enumerable, we can just assign the
// value directly and JavaScript will take care of the rest
person['fullName'] = function() {  
  return this.firstName + ' ' + this.lastName;
};

// this time, let's make man's prototype person, so all
// men share the fullName function
var man = Object.create(person);  
man['sex'] = "male";

var yehuda = Object.create(man);  
yehuda['firstName'] = "Yehuda";  
yehuda['lastName'] = "Katz";

yehuda.sex        // "male"  
yehuda.fullName() // "Yehuda Katz"  
```

## Object Literals

每次都用上面的方式设置属性依然很繁琐。JavaScript提供了 literal syntax 用来创建新对象同时设置属性。

```
var person = { firstName: "Paul", lastName: "Irish" }  
```

literal syntax 实际上是下面方式的语法糖：

```
var person = Object.create(Object.prototype);  
person.firstName = "Paul";  
person.lastName  = "Irish";  
```

注意看展开形式，literal syntax 总是把新创建对象的原型设定为 `Object.prototype`。

可是，literal syntax 只适合我们想把新对象的原型设置为 `Object.prototype` 时。但有时候，我们会想基于特定的原型创建对象：

```
var fromPrototype = function(prototype, object) {  
  var newObject = Object.create(prototype);

  for (var prop in object) {
    if (object.hasOwnProperty(prop)) {
      newObject[prop] = object[prop];      
    }
  }

  return newObject;
};

var person = {  
  toString: function() {
    return this.firstName + ' ' + this.lastName;
  }
};

var man = fromPrototype(person, {  
  sex: "male"
});

var jeremy = fromPrototype(man, {  
  firstName: "Jeremy",
  lastName:  "Ashkenas"
});

jeremy.sex        // "male"  
jeremy.toString() // "Jeremy Ashkenas"  
```

来分析下 `fromPrototype` 方法。该方法的作用是：创建新的对象，同时设置一组属性，以及设置原型。首先，用 Object.create() 创建一个新的空对象，并指定原型。然后，迭代参数对象的属性，并拷贝到新对象上。

## Native Object Orientation

到这一步，我们可以很明显的看出，原型是可以用来实现继承的。为了方便以这种方式使用，JavaScript 提供了一个 `new` 操作符。

为了方便面向对象编程，JavaScript 允许你调用一个函数对象，它作为创建新对象的原型和构造函数的结合体。

```
var Person = function(firstName, lastName) {  
  this.firstName = firstName;
  this.lastName = lastName;
}

Person.prototype = {  
  toString: function() { return this.firstName + ' ' + this.lastName; }
}
```

上面的代码创建了一个函数对象，它既是一个构造函数，也可以作为新对象的原型。接下来实现一个基于 `Person` 对象创建新对象的函数：

```
function newObject(func) {  
  // get an Array of all the arguments except the first one
  var args = Array.prototype.slice.call(arguments, 1);

  // create a new object with its prototype assigned to func.prototype
  var object = Object.create(func.prototype);

  // invoke the constructor, passing the new object as 'this'
  // and the rest of the arguments as the arguments
  func.apply(object, args);

  // return the new object
  return object;
}

var brendan = newObject(Person, "Brendan", "Eich");  
brendan.toString() // "Brendan Eich"  
```

JavaScript 的 `new` 操作符，本质上做的是同样的工作，提供了一个类似传统面向对象语言的语法。

```
var mark = new Person("Mark", "Miller");  
mark.toString() // "Mark Miller"  
```

从本质上来说，一个 JavaScript ‘类’ 就是一个函数对象，它作为构造函数，同时附加了一个原型对象。然而，早期版本的 JavaScript 并没有 `Object.create` ，但是又很需要这个方法，所以人们经常使用 `new` 操作符来实现类似功能：

```
var createObject = function (o) {  
  // we only want the prototype part of the `new`
  // behavior, so make an empty constructor
  function F() {}

  // set the function's `prototype` property to the
  // object that we want the new object's prototype
  // to be.
  F.prototype = o;

  // use the `new` operator. We will get a new
  // object whose prototype is o, and we will
  // invoke the empty function, which does nothing.
  return new F();
};
```


原文地址：[http://yehudakatz.com/2011/08/12/understanding-prototypes-in-javascript/](http://yehudakatz.com/2011/08/12/understanding-prototypes-in-javascript/)
