---
layout: post
category : javascript
title: "理解 javascript 的流程控制和错误处理"
tagline: "Supporting tagline"
tags : []
---


## 条件判断语句
### if 语句

```
if (i>25){
	console.log('25')
}else if(i>30){
	console.log('30')
}else{
	console.log('40')
}
```

ECMAScript 会自动调用Boolean()转换函数将条件表达式的结果转换为布尔值

### switch 语句

```
switch（expression）{
	case value:
		statement;
		break;
	case value:
		statement;
		break;
	default:
		statement;
}
```

每一个 case 的含义是，如果表达式的值等于这个值 value，则执行后面的 statement。break 会跳出 switch 语句。表达式不匹配任何一种情形时会执行 default 语句。

可以在 switch 语句中使用任何数据类型。每个 case 的值不一定是常量，可以是变量，也可以是表达式。

switch 语句在比较时使用的是全等操作符，不会发生类型转换。

## 循环语句


### do-while 语句

```
do{
	console.log('do')
}while(expression)
```

### while 语

```
while(expression)
	statement
```
	
### for 语句

```
for(initialization;expression;post-loop-expression)
	statement
```

由于ECMAScript中不存在块级作用域，所以在循环内部定义的变量在外部也可以访问到。

for 语句中的初始化表达式，控制表达式和循环后表达式都是可选的。

### for-in 语句

for-in 语句可以用来枚举对象的属性。

```
for (property in expression)
	statement
	
```
ECMAScript 对象的属性是没有顺序的，所以for-in语句循环的属性的顺序是不可预测的。

建议在使用for-in之前，先检查对象的值是否是null或是undefined。

### break 和 continue 语句

break语句会立即退出循环，强制继续执行循环后面的语句。
continue语句也是立即退出循环，但是退出之后会从循环的顶部继续执行。


## 异常处理语句

## promise

参考资料：

- [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Control_flow_and_error_handling](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Control_flow_and_error_handling)