---
layout: post
category : lessons
title: "Javascript 正则表达式"
tagline: "Supporting tagline"
tags : [JavaScript,RegExp]
---

## 基础知识
正则表达式模式（pattern）由简单字符（例如：`/abc/`）或是简单字符加上特殊字符（例如：`/ab*c/`）组成。

### 简单模式

简单模式由想要直接匹配的字符组成。例如：`/abc/`

### 特殊字符

- `\`
- `^` 匹配输入的起始位置。如果设置了多行标记，在换行符之后也会立即匹配。
- `$` 匹配输入的结束位置。如果设置了多行标记，在换行符之前也会立即匹配。
- `*` 前面的表达式出现 0 次或多次时匹配。等于 `{0,}`
- `+` 前面的表达式出现 1 次或多次时匹配。等于 `{1,}`
- `?` 前面的表达式出现 0 次或 1 次时匹配。等于 `{0,1}`。如果直接用在其他量词符之后，会使量词符 nongreedy（尽可能匹配少的字符）
- `.` 匹配单个字符，除了 newline character
- `(x)` 匹配 `x` 并记住该匹配。这种括号称为 capturing parentheses。
- `(?:x)` 匹配 `x` 但是不记住该匹配。这种括号称为  non-capturing parentheses。用于定义子表达式。
- `x(?=y)` 只有当 `x` 后面是 `y` 时才匹配 `x`。这称为前瞻（lookahead）
- `x(?!y)` 只有当 `x` 后面不是 `y` 时才匹配 `x`。称为否定前瞻（negated lookahead）
- `x|y` 匹配 `x` 或 `y`
- `{n}` 前面的表达式出现 `n` 次时匹配。`n` 必须是正整数。
- `{n,m}` 前面表达式最少出现 `n` 次，最多出现 `m` 次时匹配。当 `m` 省略时等于无穷大。
- `[xyz]` 字符集。匹配中括号任意一个字符。
- `[^xyz]`
- `[\b]` 匹配退格键（backspace）.
- `\b` 
- `\B`
- `\cX`
- `\d` 匹配一个数字。等于[0-9]
- `\D` 匹配任何非数字。等于[^0-9]
- `\f`
- `\n`
- `\r`
- `\s`
- `\S`
- `\t`
- `\v`
- `\w`
- `\W`
- `\n`
- `\0`
- `\xhh`
- `\uhhhh`
- `\u{hhhh}`

### 使用括号

## 创建

创建正则表达式有两种方式：

- 正则表达式字面量(regular expression literal)
- 调用 RegExp 对象的构造函数

### 正则表达式字面量

```
var expression = /pattern/flags
```

脚本加载之后，正则表达式字面量会编译正则表达式。如果模式保持不变，使用字面量的性能要好于使用构造函数。

### 调用 RegExp 对象的构造函数
构造函数接收两个参数：

- 要匹配的字符串模式
- 可选的标志字符串。

要注意的是：传给RegExp构造函数的两个参数都是字符串（不能把正则表达式字面量传递给构造函数）。

```
var pattern = new RegExp("[bc]at","i")
```

RegExp 构造函数为正则表达式提供了运行时编译。如果模式会发生变化或模式来自其它输入源（例如：用户输入）时，请使用构造函数创建正则表达式。

模式中使用的所有元字符都需要转义。元字符包括：

```
([{\^$|)?*+.]}
```

由于RegExp构造函数的模式参数是字符串，所以在某些情况下要对字符进行双重转义。所有元字符都必须双重转义，那些已经转移过的字符也是如此。



## 使用正则表达式
正则相关方法：

- exec
- test

字符串相关方法：

- match
- replace
- search
- split


## 常用正则表达式

参考资料：

- [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions)