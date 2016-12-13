---
layout: post
category : javascript
title: "理解 javascript 中的运算符"
tagline: "Supporting tagline"
tags : []
---

## 逻辑运算符
### 逻辑非
逻辑非操作符用叹号表示 `!`

可以应用于任何数据类型，逻辑非操作符首先会把操作数转换为布尔值，然后再求反。

如果操作数为空字符串、0、null、NaN、undefined，返回 true。

`!!` 构造是一个可以将任意 JavaScript 表达式转化为其等效布尔值得简单方式。
`!!variable` 的结果与 `Boolean(variable)` 相同。

逻辑运算符通常用于布尔型（逻辑）值；这种情况，它们返回一个布尔型值。然而，`&&` 和 `||` 运算符实际上返回一个指定操作数的值，因此这些运算符也用于非布尔型，它们返回一个非布尔型值。

逻辑运算符属于短路操作，如果第一个操作数能决定结果，那么就不会对第二个操作数求值。
### 逻辑与

如果第一个操作数求值结果为 false，就不会对第二个操作数求值了。

```
expr1 && expr2
```

如果 expr1 能转换成 false 则返回 expr1，否则返回 expr2。

### 逻辑或

如果第一个操作数求值结果为 true，就不会对第二个操作数求值了。

```
expr1 || expr2
```
如果 expr1 能转换成 true 则返回 expr1，否则返回 expr2。 



```
var myObject = preferredObject || backupObject
```

 
## 条件(三元)运算符

```
variable = bool_expression ? true_value : false_value;
```

基于对 `bool_expression` 求值的结果，决定给变量 `variable` 赋什么值。如果值为 `true`，则赋值为 `true_value`，否则赋值为 `false_value`。

## 逗号运算符
可以在一条语句中执行多个操作，例如：

```
var num1=1, num2=2, num3=3;
```

运算符优先级 https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Operator_Precedence

