---
layout: post
category : lessons
title: "javascript 数组循环"
tagline: "Supporting tagline"
tags : [javascript]
---

原文地址：[http://stackoverflow.com/questions/9329446/for-each-over-an-array-in-javascript](http://stackoverflow.com/questions/9329446/for-each-over-an-array-in-javascript)

##数组

1. 使用forEach(ES5+)
2. 使用for
3. 使用for-in
4. 使用for-of (隐式的使用iterator)(ES6+)
5. 显式的使用iterator

###使用forEach 和相关的

如果你的浏览器支持ECMAScript5，可以直接用forEach
```
var a=["a","b","c","d"];
a.forEach(function(entry){
    console.log(entry);
})
```
forEach 接受一个迭代函数，和一个可选的值，在迭代函数中作为this。迭代函数可以接受三个参数：每一个输入的值，输入值的位置，和数组的引用。

forEach 的好处是在循环里无需自己声明index 和value，它们以迭代函数的参数形式提供。

如果你大可不必担心性能，详情：[http://blog.niftysnippets.org/2012/02/foreach-and-runtime-cost.html](http://blog.niftysnippets.org/2012/02/foreach-and-runtime-cost.html)

ES5还提供了好几个方便操作数组的函数：

- every 当迭代函数第一次返回falsey值的时候停止循环
- some  当迭代函数第一次返回truthy值的时候停止循环
- filter    创建一个新的数组，包含那些在迭代函数中返回true的元素，去掉了返回false 的元素
- map   创建了一个新的数组，由迭代函数返回值组成
- reduce    
- reduceRight

###使用简单的for循环
有时候最古老的方式就是最好的：

```
var index;
var a=['a','b','c'];

for(index=0;index<a.length;++index){
    console.log(a[index]);
}

```

如果再循环中数组的长度不会变，而且关注性能的话，可以写的稍微复杂一些：

```
var index,len;
var a=['a','b','c']
for (index=0;len=a.length;index<len;++)
```

###正确的使用for-in

别人可能会告诉你使用for-in ,但[for-in 并不是用来做这项工作的](http://blog.niftysnippets.org/2010/11/myths-and-realities-of-forin.html)

for-in 用来循环对象的可枚举的属性，而不是数组。但是，还是很有用的，尤其是sparse array，但是需要你做好安全检查。

###使用 for-of

ES6 给javascript添加了iterators。使用iterators最简单的方式就是使用for-of 语句。例如：

```
var val;
var a=['a','b','c'];
for(val of a){
	console.log(val)
}
```

底层，它使用了iterator

###显式的使用iterator

例如：

```
var a=['a','b','c'];
var it=a.values();
var entry;
while(!(entry=it.next()).done){
	console.log(entry.value);
}
```



##Array-like Object

还有一些类数组对象，他们有length属性，和properties with numeric names，例如NodeList 实例，arguments 对象。该如何循环它的内容呢？


###使用forEach（ES5+）

假设你想在Node的childNodes 属性上使用forEach，例如：

```
Array.prototype.forEach.call(node.childNodes,function(child){
	//do something whth child
})
```

如果你要频繁使用，可以保存一个引用：

```
var forEach=Array.prototype.forEach;

forEach.call(node.childNodes,function(child){
	//do something
})
```

###使用for循环
###正确的使用for-in 
也要做和数组一样的安全检查
###使用for-of(ES6)
###显式的使用iterator

###创建一个真数组
把类数组对象转换成真数组。利用数组的slice方法。

```
var trueArray=Array.prototype.slice.call(arrayLikeObject,0);
```

实例：

```
var divs = Array.prototype.slice.call(document.querySelectorAll("div"), 0);
```

##Caveat for host-provided objects



























