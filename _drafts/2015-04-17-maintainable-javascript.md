---
layout: post
category : lessons
title: "编写可维护的javascript"
tagline: "Supporting tagline"
tags : [javascript]
---

##直接量
###null
把null 理解为对象的占位符。

可以使用null的场景：

- 用来初始化变量，该变量可能赋值为一个对象
- 用来和一个已经初始化的变量比较，这个变量可以是也可以不是一个对象？？？？？
- 当函数的参数值期望的是对象时，用作参数传入
- 当函数的返回值期望是对象时，用作返回值传出

不应该使用null的场景：

- 不要使用null来检测是否传入了某个参数
- 不要用null来检测一个未初始化的变量

要避免"空比较"，因为仅仅和null比较并不能提供足够的信息来判断后续代码的执行是否真的安全。
###undefined

没有被初始化的变量都有一个初始值，即undefined，表示该变量等待被赋值。

要避免在代码中使用undefined。

注意：

- null==undefined  为true
- 不管是未初始化的变量，还是未声明的变量，typeof的运算结果都是"undefined"



###对象直接量
好的写法：

```
var book={ title:"javascript", author: "zakas"}
```
不好的写法：

```
var book=new Object();
book.title="javascript"
```

###数组直接量
好的写法：

```
var color=["red","green","blue"]
```

不好的写法：

```
var color=new Array('red','green','blue')
```

##类型检测

###检测原始值
javascript有5种原始类型：字符串、数字、布尔值、null、undefined。

如果你希望一个值是字符串、数字、布尔值、undefined。最佳选择是typeof。对应的返回值为"string","number","boolean","undefined"

```
typeof varialbe
```

最后一个原始值，null，一般不应用于检测语句。但有一个例外，如果所期望的值真的为null，可以使用=== 或者!== 来和null来比较。

typeof null 返回"object"，不要使用这种方式，直接用恒等运算符。

###检测引用值

javascript除了原始值之外都是引用值。包括Object、Array、Date和Error。typeof 这些引用类型时，都会返回"object"

检测引用值使用instanceof

####检测函数
####检测数组

ECMAScript 5

```
Array.isArray()
```
###检测属性


