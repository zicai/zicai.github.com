---
layout: post
category : lessons
title: "javascript better practice"
tagline: "Supporting tagline"
tags : [javascript]
---

## null
把 null 理解为对象的占位符。

可以使用 null 的场景：

- 用来初始化变量，该变量可能赋值为一个对象
- 用来和一个已经初始化的变量比较，这个变量可以是也可以不是一个对象？？？？？
- 当函数的参数值期望的是对象时，用作参数传入
- 当函数的返回值期望是对象时，用作返回值传出

不应该使用 null 的场景：

- 不要使用 null 来检测是否传入了某个参数
- 不要用 null 来检测一个未初始化的变量

要避免"空比较"，因为仅仅和 null 比较并不能提供足够的信息来判断后续代码的执行是否真的安全。
## undefined

没有被初始化的变量都有一个初始值，即 undefined，表示该变量等待被赋值。

要避免在代码中使用 undefined。

注意：

- null == undefined 为 true
- 不管是未初始化的变量，还是未声明的变量，typeof 的运算结果都是 "undefined"


## 字符串

### 切割字符串

- 按长度切割：
	
		str.substr(start[, length])
		
- 按位置切割：

		str.slice(beginSlice[, endSlice])
		str.substring(indexStart[, indexEnd])
		
	[slice 和 substring 的区别](http://stackoverflow.com/questions/2243824/what-is-the-difference-between-string-slice-and-string-substring)
     
    上面三个方法，如果省略第二个参数，默认都是切割到字符串末尾

### 匹配字符串

- 查找子字符串：

		str.indexOf(searchValue[, fromIndex])
		str.lastIndexOf(searchValue[, fromIndex])
	
	如果没找到返回 -1

- 按正则匹配：
	
	- 只想知道是否匹配：`regexObj.test(str)`
	- 想知道是否匹配，同时要获取位置：`str.search(regexp)`
	- 更多信息：`str.match(regexp)`

## 数字

### 取整

- 向上取整：`Math.ceil()`
- 向下取整：`Math.floor()`
- 四舍五入：`Math.round()`

### 获取随机数

```
Math.random() 返回 [0, 1) 的伪随机数

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

## 数组

- 新建数组

		var color=["red","green","blue"]

- 拼接为字符串

		arr.toString() // 逗号分割
		arr.join([separator = ',']) // 指定分隔符
		
		
- 迭代数组

	ECMAScript 5为数组定义了五个迭代方法。每个方法都接收两个参数：要在每一项上运行的函数和运行该函数的作用域对象（可选）。第二个参数会影响`this`的值。第一个参数：函数，接受三个参数：数组项的值，该项在数组中的位置和数组对象本身。

	- 过滤数组

			filter() // 返回该函数会返回`true`的项组成的数组
		
	- 判断数组

			every() // 如果该函数对每一项都返回`true`,则返回`true`
			some() // 如果该函数对任一项返回`true`,则返回`true`
	
	- 操作数组项

			forEach() // 没有返回值
			
	- 映射数组

			map() // 返回每次函数调用的结果组成的数组
		
- 在数组中查找项

		arr.indexOf(searchElement[, fromIndex = 0])
		arr.lastIndexOf(searchElement[, fromIndex = arr.length - 1])
		
- 增加、删除项

		shift() // 方法移除数组中的第一项，并返回移除的项。
		unshift() // 在数组前端添加任意个项，并返回新数组的长度。
		pop() // 从数组末尾移除最后一项，返回移除的项。
		push() // 在数组尾部添加任意个项，并返回修改后的数组长度。
		
- 数组排序

		reverse() // 方法会反转数组项的排序
		sort() // 默认是用比较字符串的方式按升序排列，也可以自定义排序函数。
		
- 拼接数组

		var new_array = old_array.concat(value1[, value2[, ...[, valueN]]])
		
- 切割数组

		arr.slice([begin[, end]]) // 返回起始位置和结束位置之间的项（不包括结束位置）。
		
## 对象

- 新建对象

		var book={ title:"javascript", author: "zakas"}
		
		
		
## Date
时间戳是自1970 年1 月1 日（00:00:00 GMT）以来的秒数。它也被称为Unix 时间戳（Unix Timestamp）。

- 新建日期对象

		new Date()
		new Date(value) // value 为毫秒时间戳
		new Date(year, month[, day[, hour[, minutes[, seconds[, milliseconds]]]]]); // 注意月份是从 0 开始的
		
	新建日期对象时必须按照构造函数方式使用 `new` ,如果以常规方法方式调用 `Date()` ，那么只会返回字符串，而不是日期对象。和其它 JavaScript 对象不同，日期对象没有 literal syntax。
	
- 获取毫秒数
	
		Date.now() // 返回当前时间的毫秒数。
		dateObj.valueOf() // 返回特定时间的毫秒数。

- 获取字符串表示

		dateObj.totoISOString() // 格式：2016-01-20T02:32:35.959Z
		a.toLocaleString() // 格式：2016/1/20 上午10:32:35
		a.toLocaleDateString() // 格式：2016/1/20
		
## 正则

- 新建正则

		var expression=/pattern/flags
		var pattern=new RegExp("[bc]at","i") // 传给RegExp构造函数的两个参数都是字符串（不能把正则表达式字面量传递给构造函数）

	模式中使用的所有元字符都需要转义。元字符包括：


		（[{\^$|)?*+.]}


	由于RegExp构造函数的模式参数是字符串，所以在某些情况下要对字符进行双重转义。所有元字符都必须双重转义，那些已经转移过的字符也是如此。


## 类型检测

### 检测原始值
javascript有5种原始类型：字符串、数字、布尔值、null、undefined。

如果你希望一个值是字符串、数字、布尔值、undefined。最佳选择是typeof。对应的返回值为"string","number","boolean","undefined"

```
typeof varialbe
```

最后一个原始值，null，一般不应用于检测语句。但有一个例外，如果所期望的值真的为null，可以使用=== 或者!== 来和null来比较。

typeof null 返回"object"，不要使用这种方式，直接用恒等运算符。

### 检测引用值

javascript除了原始值之外都是引用值。包括Object、Array、Date和Error。typeof 这些引用类型时，都会返回"object"

检测引用值使用instanceof

#### 检测函数
#### 检测数组

ECMAScript 5

```
Array.isArray()
```
### 检测属性
使用in运算符

```
if("count" in object){
	a=object.count
}
```

注意：在coffeescript中用in判断数据在数组中是否出现, 而 of 可以探测 JavaScript 对象的属性是否存在.
