---
layout: post
category : lessons
title: "coffeescript"
tagline: ""
tags : [coffeescript]
---
功能array comprehensions, prototype aliases and classes
##语法
没有分号。使用缩进来代替花括号。

###注释
注释格式与ruby一致，以井号开头

```
# A comment
```

支持多行注释，多行注释会添加到生成的javascript中

```
###
	A multiline comment
###
```

###变量与作用域
编译时，CoffeeScript使用一个匿名函数把所有脚本都包裹起来，将其限定在局部作用域中，并且为所有的变量赋值前自动添加var。

可以通过下面的方式创建全局变量

```
exports = this
exports.MyVariable = "foo-bar" 
```


###数组和对象
花括号可以省略，分割用的逗号可以通过换行省略。中括号不能省略。


```
object1 = {one: 1, two: 2}

# Without braces
object2 = one: 1, two: 2

# Using new lines instead of commas
object3 = 
  one: 1
  two: 2
  
song = ["do", "re", "mi", "fa", "so"]

bitlist = [
  1, 0, 1
  0, 0, 1
  1, 1, 0
]

```




###函数
函数通过一组可选的圆括号包裹的参数, 一个箭头, 一个函数体来定义. 一个空的函数像是这样:  `->`
。函数可以是一行，也可以是多行。函数的最后一个表达式回作为隐式的返回值。

```
square=(x) -> x*x
```
####参数默认值
一些函数函数参数会有默认值, 当传入的参数的不存在 (null 或者 undefined) 时会被使用.

```
fill = (container, liquid = "coffee") ->
  "Filling the #{container} with #{liquid}..."
```

####变参
使用`...`表示

```
sum = (nums...) -> 
  result = 0
  nums.forEach (n) -> result += n
  result 
```

nums 不是一个arguments 对象，而是一个真数组。

####函数调用
无参数调用函数时需要加括号。如果函数被至少一个参数跟着的话，coffeescript会自动调用这个函数。

####函数上下文
使用胖箭头`=>`，可以确保函数的上下文绑定为当前的上下文。

```
this.clickHandler = -> alert "clicked"
element.addEventListener "click", (e) => this.clickHandler(e)
```

这种绑定的思想与jQuery的 `proxy()`或者ES5's的`bind()`函数是类似的概念。

###流程控制
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

coffeescript支持一项ruby特性，在if语句之前使用前缀表达式。

```
mood = greatlyImproved if singing
```
unless 是if的否定


####三元表达式
CoffeeScript 里不存在直白的三元表达式. — 你只要在一行内使用普通的 if 语句。在单行if语句中必须使用then关键字，这样coffeescript才能知道执行体从哪里开始。

```
date = if friday then sue else jill
```

####存在操作符
CoffeeScript存在操作符?只会在变量为null或者undefined的时候会返回真，与Ruby的nil?类似。


###操作符和别名

coffeescript|javascript
|---|---|
is|===
isnt|!==
not|!
and |&&
or|||
true,yes,on|true
false,no,off|false
@,this|this
of |in
in|no js equivalent
::|prototype

###字符串插值

在双引号的字符串中可以包含#{}标记，用来插入表达式。

```
favourite_color = "Blue. No, yel..."
question = "Bridgekeeper: What... is your favourite color?
            Galahad: #{favourite_color}
            Bridgekeeper: Wrong!
            " 
```

允许多行字符串，无需`+`
###循环和列表解析

JavaScript中的数组迭代使用一种相当古老的语法，看上去更像一个类似于C之类的老语言，而不是现代的面向对象的语言。ES5引入forEach()函数来稍微改善了下这种情况，但是这样的话每次迭代都需要调用一次函数，因此运行速度会变慢。再一次，CoffeeScript给出一种漂亮的语法拯救了我们：

```
for name in ["Roger", "Roderick", "Brian"]
  alert "Release #{name}" 
```
如果你需要知道当前迭代索引的话，只需要再多传一个参数：

```
for name, i in ["Roger the pickpocket", "Roderick the robber"]
  alert "#{i} - Release #{name}" 
```
使用前缀的形式你可以一行代码完成迭代。

```
release prisoner for prisoner in ["Roger", "Roderick", "Brian"] 
```
就如Python的推导式一样，你可以过滤它们：

```
prisoners = ["Roger", "Roderick", "Brian"]
release prisoner for prisoner in prisoners when prisoner[0] is "R" 
```
你可以使用推导式来迭代对象的全部属性，不过要使用of代替in关键字。

```
names = sam: seaborn, donna: moss
alert("#{first} #{last}") for first, last of names 
```
唯一CoffeeScript暴露出来的底层循环语法是while循环。它与原JavaScript中while循环的行为差不多，只是包含了已添加的优点，它能返回一个结果数组。看起来像Array.prototype.map()函数。

```
num = 6
minstrel = while num -= 1
  num + " Brave Sir Robin ran away" 
```
























