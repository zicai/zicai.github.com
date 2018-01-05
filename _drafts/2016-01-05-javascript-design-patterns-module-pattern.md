---
layout: post
category : lessons
title: "JavaScript 设计模式--module pattern"
tagline: "Supporting tagline"
tags : [javascript]
---



在 JavaScript 中，使用模块有很多种选择，包括：

- The module pattern
- Object literal notation
- AMD modules
- CommonJS modules
- ECMAScript Harmony modules

Module pattern 部分是基于 object literal，所以，我们先来看object literal

## Object literal

```javascript
var myObjectLiteral = {
 
    variableKey: variableValue,
 
    functionKey: function () {
      // ...
    }
};
```

一个更完整的例子：

```javascript
var myModule = {
 
  myProperty: "someValue",
 
  // object literals can contain properties and methods.
  // e.g we can define a further object for module configuration:
  myConfig: {
    useCaching: true,
    language: "en"
  },
 
  // a very basic method
  saySomething: function () {
    console.log( "Where in the world is Paul Irish today?" );
  },
 
  // output a value based on the current configuration
  reportMyConfig: function () {
    console.log( "Caching is: " + ( this.myConfig.useCaching ? "enabled" : "disabled") );
  },
 
  // override the current configuration
  updateMyConfig: function( newConfig ) {
 
    if ( typeof newConfig === "object" ) {
      this.myConfig = newConfig;
      console.log( this.myConfig.language );
    }
  }
};
 
// Outputs: Where in the world is Paul Irish today?
myModule.saySomething();
 
// Outputs: Caching is: enabled
myModule.reportMyConfig();
 
// Outputs: fr
myModule.updateMyConfig({
  language: "fr",
  useCaching: false
});
 
// Outputs: Caching is: disabled
myModule.reportMyConfig();
```

使用 object literal 可以协助封装和组织你的代码。Rebecca Murphey 有一篇关于 object literal 的[深度解析](Rebecca Murphey)

module pattern 仍然是使用 object literal，只不过是将其作为一个 scoping function 的返回值。

## module pattern

例子：

```javascript
var testModule = (function () {
 
  var counter = 0;
 
  return {
 
    incrementCounter: function () {
      return counter++;
    },
 
    resetCounter: function () {
      console.log( "counter value prior to reset: " + counter );
      counter = 0;
    }
  };
 
})();
 
// Usage:
 
// Increment our counter
testModule.incrementCounter();
 
// Check the counter value and reset
// Outputs: counter value prior to reset: 1
testModule.resetCounter();
```

模板

```javascript
var myNamespace = (function () {
 
  var myPrivateVar, myPrivateMethod;
 
  // A private counter variable
  myPrivateVar = 0;
 
  // A private function which logs any arguments
  myPrivateMethod = function( foo ) {
      console.log( foo );
  };
 
  return {
 
    // A public variable
    myPublicVar: "foo",
 
    // A public function utilizing privates
    myPublicFunction: function( bar ) {
 
      // Increment our private counter
      myPrivateVar++;
 
      // Call our private method using bar
      myPrivateMethod( bar );
 
    }
  };
 
})();
```

优势：

### module pattern 变种

#### Import mixin
将全局变量作为参数传入

```javascript
// Global module
var myModule = (function ( jQ, _ ) {
 
    function privateMethod1(){
        jQ(".container").html("test");
    }
 
    function privateMethod2(){
      console.log( _.min([10, 5, 100, 2, 1000]) );
    }
 
    return{
        publicMethod: function(){
            privateMethod1();
        }
    };
 
// Pull in jQuery and Underscore
})( jQuery, _ );
 
myModule.publicMethod();
```

#### Exports

```javascript
// Global module
var myModule = (function () {
 
  // Module object
  var module = {},
    privateVariable = "Hello World";
 
  function privateMethod() {
    // ...
  }
 
  module.publicProperty = "Foobar";
  module.publicMethod = function () {
    console.log( privateVariable );
  };
 
  return module;
 
})();
```

### 特定框架的 module pattern 的实现

- dojo
- extjs
- yui
- jquery


### 优势
### 劣势


## The Revealing Module Pattern
a slightly improved version - Christian Heilmann’s Revealing Module pattern.

### 优势
### 劣势




参考资源：

- https://addyosmani.com/resources/essentialjsdesignpatterns/book/#modulepatternjavascript
- [深入解读 module pattern](http://www.adequatelygood.com/JavaScript-Module-Pattern-In-Depth.html)


















