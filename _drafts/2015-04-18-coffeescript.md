---
layout: post
category : lessons
title: "coffeescript"
tagline: ""
tags : [coffeescript]
---

##数组和对象
对象和数组的字面量看起来很像在 JavaScript 中的写法。如果单个属性被写在自己的一行里, 那么逗号是可以省略的.

```
song = ["do", "re", "mi", "fa", "so"]

singers = {Jagger: "Rock", Elvis: "Roll"}

bitlist = [
  1, 0, 1
  0, 0, 1
  1, 1, 0
]

kids =
  brother:
    name: "Max"
    age:  11
  sister:
    name: "Ida"
    age:  9
```




##函数
函数通过一组可选的圆括号包裹的参数, 一个箭头, 一个函数体来定义. 一个空的函数像是这样:  `->`

```
square=(x) -> x*x
```
###参数默认值
一些函数函数参数会有默认值, 当传入的参数的不存在 (null 或者 undefined) 时会被使用.

```
fill = (container, liquid = "coffee") ->
  "Filling the #{container} with #{liquid}..."
```

##条件语句
if/else 表达式可以不用圆括号和花括号就写出来. 就像函数和其他块级表达式那样, 多行的条件可以通过缩进来表明. 

```
if happy and knowsIt
  clapsHands()
  chaChaCha()
else if
  showIt()
else
  bongbong()
```
###条件赋值

```
mood = greatlyImproved if singing
```
###三元表达式
CoffeeScript 里不存在直白的三元表达式. — 你只要在一行内使用普通的 if 语句

```
date = if friday then sue else jill
```



