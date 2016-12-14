---
layout: post
category : JavaScript
title: "Underscore 源码学习"
tagline: "Supporting tagline"
tags : [Underscore]
---
##集合

Note: Collection functions work on arrays, objects, and array-like objects such as arguments, NodeList and similar. But it works by duck-typing, so avoid passing objects with a numeric length property. It's also good to note that an each loop cannot be broken out of — to break, use _.find instead.

###遍历
each
map
pluck
invoke
###查找
find

where
findWhere

###过滤
filter
reject
partition
###判定
every
some
contains

###随机
shuffle
sample

###其它

reduce
reduceRight
max
min
size

sortBy
groupBy
indexBy
countBy
toArray



##数组

###去掉数组中部分元素
without
difference
compact
uniq
###返回指定位置元素
first
initial
last
rest
###多数组操作
union
intersection
difference
zip
unzip
###数组转对象
object

###查找索引
indexOf
lastIndexOf
sortedIndex
findIndex
findLastIndex

###创建数组
range
###其它
flatten


###数组变对象


##对象
###获取属性
keys
allKeys
findkey
has
###获取属性值
values
property
propertyOf
###转换属性值
mapObject
###键值对换
invert
###对象变数组
pairs
###获取对象所有方法
functions

###扩展对象
extend
extendOwn
###精简对象
pick
omit

## 判断
### 判断数据类型
* isElement
* isArray
* isObject
* isArguments
* isFunction
* isString
* isNumber
* isBoolean
* isDate
* isRegExp
* isError
* isNaN
* isNull
* isUndefined

### 一些用于比较的函数

* matcher
* isEqual
* isMatch
* isEmpty
* isFinite

###其它
defaults
clone
tap


## 源码学习

源码版本：1.8.3

首先是环境初始化，定义了一些别名

```
// Baseline setup
// --------------

// Establish the root object, `window` in the browser, or `exports` on the server.
var root = this;

// Save the previous value of the `_` variable.
var previousUnderscore = root._;

// Save bytes in the minified (but not gzipped) version:
var ArrayProto = Array.prototype,
  ObjProto = Object.prototype,
  FuncProto = Function.prototype;

// Create quick reference variables for speed access to core prototypes.
var
  push = ArrayProto.push,
  slice = ArrayProto.slice,
  toString = ObjProto.toString,
  hasOwnProperty = ObjProto.hasOwnProperty;

// All **ECMAScript 5** native function implementations that we hope to use
// are declared here.
var
  nativeIsArray = Array.isArray,
  nativeKeys = Object.keys,
  nativeBind = FuncProto.bind,
  nativeCreate = Object.create;

// Naked function reference for surrogate-prototype-swapping.
var Ctor = function() {};

// Create a safe reference to the Underscore object for use below.
var _ = function(obj) {
  if (obj instanceof _) return obj;
  if (!(this instanceof _)) return new _(obj);
  this._wrapped = obj;
};

```

导出与版本号

```
// Export the Underscore object for **Node.js**, with
// backwards-compatibility for the old `require()` API. If we're in
// the browser, add `_` as a global object.
if (typeof exports !== 'undefined') {
  if (typeof module !== 'undefined' && module.exports) {
    exports = module.exports = _;
  }
  exports._ = _;
} else {
  root._ = _;
}

// Current version.
_.VERSION = '1.8.3';
```

### 一些内部函数

```
// Internal function that returns an efficient (for current engines) version
// of the passed-in callback, to be repeatedly applied in other Underscore
// functions.
var optimizeCb = function(func, context, argCount) {
  if (context === void 0) return func;
  switch (argCount == null ? 3 : argCount) {
    case 1: return function(value) {
      return func.call(context, value);
    };
    case 2: return function(value, other) {
      return func.call(context, value, other);
    };
    case 3: return function(value, index, collection) {
      return func.call(context, value, index, collection);
    };
    case 4: return function(accumulator, value, index, collection) {
      return func.call(context, accumulator, value, index, collection);
    };
  }
  return function() {
    return func.apply(context, arguments);
  };
};

// A mostly-internal function to generate callbacks that can be applied
// to each element in a collection, returning the desired result — either
// identity, an arbitrary callback, a property matcher, or a property accessor.
var cb = function(value, context, argCount) {
  if (value == null) return _.identity;
  if (_.isFunction(value)) return optimizeCb(value, context, argCount);
  if (_.isObject(value)) return _.matcher(value);
  return _.property(value);
};
_.iteratee = function(value, context) {
  return cb(value, context, Infinity);
};

...

// Keep the identity function around for default iteratees.
_.identity = function(value) {
  return value;
};
```

先来看 `_.identity`，它返回参数值。对应数学中的：f(x)=x。看起来没啥用，实际上在整个 underscore 中都作为默认 iteratee。

再看 `_.iteratee(value, [context])`，它用来生成一个可以应用到集合中每个元素的回调函数。对于常用场景有几种缩写方式。根据 value 不同，返回值也不同

```
// No value
_.iteratee();
=> _.identity()

// Function
_.iteratee(function(n) { return n * 2; });
=> function(n) { return n * 2; }

// Object
_.iteratee({firstName: 'Chelsea'});
=> _.matcher({firstName: 'Chelsea'});

// Anything else
_.iteratee('firstName');
=> _.property('firstName');
```

### 判断数据类型相关的函数

#### isObject

```
// Is a given variable an object?
_.isObject = function(obj) {
  var type = typeof obj;
  return type === 'function' || type === 'object' && !!obj;
};
```
说明：

- 使用 `typeof obj`，当 `obj` 为对象或是 `null`，都会返回 `'object'`；当 `obj` 为函数时，返回 `'function'`
- 运算符优先级：逻辑非 高于 全等 高于 逻辑与 高于 逻辑或
- `!!obj` 的结果与 `Boolean(obj)` 相同，用来将任意 JavaScript 表达式转化为其等效布尔值
- todo: 为什么不用 instanceof 或者 toString

#### isArray

```
// Is a given value an array?
// Delegates to ECMA5's native Array.isArray
_.isArray = nativeIsArray || function(obj) {
  return toString.call(obj) === '[object Array]';
};

```

说明：

- `toString` 来自 Object.prototype ，如果该方法在自定义对象中没有被重写，那么 toString() 返回 "[object type]"，其中 type 就是对象的类型。可以利用这一点来检测对象的类型，参见：[https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/toString#Using_toString()_to_detect_object_class](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/toString#Using_toString()_to_detect_object_class)

#### isArguments, isFunction, isString, isNumber, isDate, isRegExp, isError


```
// Add some isType methods: isArguments, isFunction, isString, isNumber, isDate, isRegExp, isError.
_.each(['Arguments', 'Function', 'String', 'Number', 'Date', 'RegExp', 'Error'], function(name) {
  _['is' + name] = function(obj) {
    return toString.call(obj) === '[object ' + name + ']';
  };
});

// Define a fallback version of the method in browsers (ahem, IE < 9), where
// there isn't any inspectable "Arguments" type.
if (!_.isArguments(arguments)) {
  _.isArguments = function(obj) {
    return _.has(obj, 'callee');
  };
}

// Optimize `isFunction` if appropriate. Work around some typeof bugs in old v8,
// IE 11 (#1621), and in Safari 8 (#1929).
if (typeof /./ != 'function' && typeof Int8Array != 'object') {
  _.isFunction = function(obj) {
    return typeof obj == 'function' || false;
  };
}
```

说明：

- 考虑到简单类型，所以用不能使用 instanceof 来判断

#### isBoolean

```
// Is a given value a boolean?
_.isBoolean = function(obj) {
  return obj === true || obj === false || toString.call(obj) === '[object Boolean]';
};
```

#### isNull

```
// Is a given value equal to null?
_.isNull = function(obj) {
  return obj === null;
};
```

#### isUndefined

```
// Is a given variable undefined?
_.isUndefined = function(obj) {
  return obj === void 0;
};
```

说明：

- void 操作符用来返回 undefined。

#### isNaN

```
// Is the given value `NaN`? (NaN is the only number which does not equal itself).
_.isNaN = function(obj) {
  return _.isNumber(obj) && obj !== +obj;
};
```

说明：

- 只有当 obj 是 NaN 时，_.isNaN 才返回 true。这和原生的 isNaN 不同，原生的 isNaN 函数，当参数是任何不能转换为数值的值都会返回 true。

#### isElement

```
// Is a given value a DOM element?
_.isElement = function(obj) {
  return !!(obj && obj.nodeType === 1);
};
```

说明：

- 当 obj 为简单类型时
	- 当 obj 为 null 或者 undefined 时，直接访问 obj.nodeType 是会报错的。而逻辑与运算符前面的 obj 就可以避免报错。
	- 当 obj 为 boolean、number、string，访问 obj.nodeType 是不会报错的，因为读取基本类型值时，后台会创建一个对应的基本包装类型的对象，从而可以访问属性和方法。

	
### 一些用于比较的函数（断言函数）

#### isMatch

判断对象是否包含指定的键值对

```
// Returns whether an object has a given set of `key:value` pairs.
_.isMatch = function(object, attrs) {
  var keys = _.keys(attrs), length = keys.length;
  if (object == null) return !length;
  var obj = Object(object);
  for (var i = 0; i < length; i++) {
    var key = keys[i];
    if (attrs[key] !== obj[key] || !(key in obj)) return false;
  }
  return true;
};
```

说明


#### matcher

返回一个断言函数，该函数用来判断传入对象是否包含所有 attrs 里所有键值对。

```
// Returns a predicate for checking whether an object has a given set of
// `key:value` pairs.
_.matcher = _.matches = function(attrs) {
  attrs = _.extendOwn({}, attrs);
  return function(obj) {
    return _.isMatch(obj, attrs);
  };
};
```

说明：

- 1.8.0 版本将 `_.matches` 标记为过时，推荐使用函数名 `_.matcher`
- 


#### isEqual

#### isEmpty
#### isFinite

### 对象属性名相关

#### has
判断属性是否是对象自身的（不是原型链上的）

```
// Shortcut function for checking if an object has a given property directly
// on itself (in other words, not on a prototype).
_.has = function(obj, key) {
  return obj != null && hasOwnProperty.call(obj, key);
};
```

说明：

- todo:为啥不用全等？？

#### keys

获取对象所有自身属性名

```
// Retrieve the names of an object's own properties.
// Delegates to **ECMAScript 5**'s native `Object.keys`
_.keys = function(obj) {
  if (!_.isObject(obj)) return [];
  if (nativeKeys) return nativeKeys(obj);
  var keys = [];
  for (var key in obj) if (_.has(obj, key)) keys.push(key);
  // Ahem, IE < 9.
  if (hasEnumBug) collectNonEnumProps(obj, keys);
  return keys;
};
```



#### allKeys

获取对象所有属性名，包括继承的

```
// Retrieve all the property names of an object.
_.allKeys = function(obj) {
  if (!_.isObject(obj)) return [];
  var keys = [];
  for (var key in obj) keys.push(key);
  // Ahem, IE < 9.
  if (hasEnumBug) collectNonEnumProps(obj, keys);
  return keys;
};
```

对于 IE < 9，keys 和 allKeys 都用到了下面的函数：

```
// Keys in IE < 9 that won't be iterated by `for key in ...` and thus missed.
var hasEnumBug = !{toString: null}.propertyIsEnumerable('toString');
var nonEnumerableProps = ['valueOf', 'isPrototypeOf', 'toString',
  'propertyIsEnumerable', 'hasOwnProperty', 'toLocaleString'];

function collectNonEnumProps(obj, keys) {
  var nonEnumIdx = nonEnumerableProps.length;
  var constructor = obj.constructor;
  var proto = (_.isFunction(constructor) && constructor.prototype) || ObjProto;

  // Constructor is a special case.
  var prop = 'constructor';
  if (_.has(obj, prop) && !_.contains(keys, prop)) keys.push(prop);

  while (nonEnumIdx--) {
    prop = nonEnumerableProps[nonEnumIdx];
    if (prop in obj && obj[prop] !== proto[prop] && !_.contains(keys, prop)) {
      keys.push(prop);
    }
  }
}
```

#### findKey


```
// Returns the first key on an object that passes a predicate test
_.findKey = function(obj, predicate, context) {
  predicate = cb(predicate, context);
  var keys = _.keys(obj), key;
  for (var i = 0, length = keys.length; i < length; i++) {
    key = keys[i];
    if (predicate(obj[key], key, obj)) return key;
  }
};
```

### 其它

```
// Extend a given object with all the properties in passed-in object(s).
_.extend = createAssigner(_.allKeys);
```





















