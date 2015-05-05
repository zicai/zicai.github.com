---
layout: post
category : lessons
title: "meteor spacebars"
tagline: "Supporting tagline"
tags : [meteor]
---

Spacebars is Meteor's dialect of Handlebars.

##Reactivity Model
##标示符和path
Spacebars标示符可以是javascript标示符，也可以是方括号`[ ]`括起来的字符串。还有特殊的标示符，`this`（等同于`.`） 和 `..`。

Spacebars path由一个或多个标示符组成，由`.`或`/`分隔。

### name resolution

### Path evaluation
##Helper Arguments
##Inclusion and Block Arguments
##Template Tag Placement Limitations
##Double-braced Tags
###safeString
###in Attribute Values
###Dynamic Attributes
##Triple-braced Tags
用于在模板中插入原始HTML。

被插入的Html必须包含balanced html tag。例如：你不能插入`</div><div>`来关闭一个已存在的div,然后创建一个新的
##Inclusion Tags
inclusion tag 形式为：`{{> templateName}}` 或 `{{> templateName dataObj}}`。其它形式的参数都是用来构建data对象的语法糖（参见Inclusion and Block Arguments）

inclusion tag 在当前位置插入一个给定模板的实例。如果带参数，参数会成为data context，就如同下面的代码：
```
{{#with dataObj}}
  {{> templateName}}
{{/with}}
```
除了直接指定模板名字外，inclusion tag还可以指定一个path, that evalutes to a template object,或是一个返回一个模板对象的函数。

###function returning a template
如果一个inclusion tag 解析为一个函数，那么该函数必须返回一个模板对象或是null。该函数reactively re-run，如果返回值发生变化，模板会被替换。

##Block Tag
Block tag 会调用内置的directives 或是自定义的Block helpers。模板内容会被实例化一次或多次，也可能不实例化。
```
{{#block}}
  <p>Hello</p>
{{/block}}
```
Block tag 还可以指定 “else”内容，用`{{else}}`标签与主内容分隔开

Block tag 必须包含balanced html tag.

Block tag 可以用在属性值：

```
<div class="{{#if done}}done{{else}}notdone{{/if}}">
  ...
</div>
```

##If/Unless
当值为`null,undefined,0,"",false`时视为false ，其它值视为true。
`#unless` 与 `#if`相反
##With
##Each
###reactivity Model for Each
##Custom Block Helpers
##Comment Tags
##Html Dialect
Spacebars 在运行时会校验你的HTML，如果你违反了基本的HTML语法，导致它无法确定你的HTML结构，spacebars 会抛出一个compile-time 错误。

对不符合规范的标签，spacebars 并不像浏览器那么宽容。
##Top-level Elements in a `.html` file
严格的说，`<template>`标签并不属于Spacebars模板语言。Meteor 中一个`foo.html`模板文件由下面三种元素组成：
- `<template name="myName">`：`<template>`标签包含spacebars 模板，会被编译成`Template.myName`组件
- `<head>`：如果`<head>`标签出现多次（可能是在多个文件中），`<head>`元素的内容会被串联起来。
- `<body>`：会被编译成`Template.body` 组件。如果`<body>`标签出现多次，内容会被串联起来。















