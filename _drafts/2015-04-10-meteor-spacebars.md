---
layout: post
category : lessons
title: "meteor spacebars"
tagline: "Supporting tagline"
tags : [meteor]
---

Spacebars is Meteor's dialect of Handlebars.
## 开始
Spacebars 模板由HTML标签和模板标签组成，模板标签由`{{ }}`表示。

```
<template name="myPage">
  <h1>{{pageTitle}}</h1>

  {{> nav}}

  {{#each posts}}
    <div class="post">
      <h3>{{title}}</h3>
      <div class="post-content">
        {{{content}}}
      </div>
    </div>
  {{/each}}
</template>
```

主要有四种模板标签：

- `{{pageTitle}}`: 用来插入文本，文本自动被转换成没有危害的。它可以包含任何字符(例如：`<`),并且不会生成任何HTML标签
- `{{> nav}}`: 插入其它的模板
- `{{#each }}`: Block template tag。内置的有`#if,#each,#with,#unless`。也可以自定义
- `{{{content}}}`: 用来插入原生的HTML。注意：你需要确保HTML是安全的


## Reactivity Model
Spacebars 模板可以细粒度的更新，反映数据的变化。

每一个模板标签的DOM会自动的更新。
## 标示符和path
Spacebars标示符可以是javascript标示符，也可以是方括号`[ ]`括起来的字符串。还有特殊的标示符，`this`（等同于`.`） 和 `..`。Brackets are required to use one of the following as the first element of a path: else, this, true, false, and null. Brackets are not required around JavaScript keywords and reserved words like var and for.

Spacebars path由一个或多个标示符组成，由`.`或`/`分隔。

### name resolution
path 的第一个标示符用下面两种方式解析：

- 当前数据上下文的索引。例如：标示符`foo`指向当前数据上下文的`foo`属性
- 是一个模板helper。例如：标示符指向一个当前模板可以获取的helper函数(或常量)

模板helper优先于数据上下文的属性。

如果一个path以`..`开头，则会使用外层数据上下文，外层数据上下文可能是外层的`#each,#with`，或是模板包含。
### Path evaluation

当解析一个path时，第一个标示符之后的标示符会看做当前对象的索引，就像javascript的`.`。但是，如果你试图索引一个非对象，或是一个undefined值，是不会报错的。

另外，Spacebars 会自动为你调用函数，所以`{{foo.bar}}`可能解释为`foo().bar`, `foo.bar()`, 或 `foo().bar()`
## Helper Arguments

helper 的参数可以使任意的path或是标示符，或是字符串，布尔值，number或是null。

双大括号和三大括号的模板标签可以接受任意数量的位置参数和关键字参数(positional and keyword arguments)，例如：

```
{{frob a b c verily=true}}
```

调用：

```
frob(a, b, c, Spacebars.kw({verily: true}))
```

`Spacebars.kw` 构建一个对象，是`Spacebars.kw` 的实例，它的`.hash` 属性等于它的参数。

helper的实现可以通过this获取当前数据上下文。
##Inclusion and Block Arguments

Inclusion tags (`{{> foo}}`) 和 block tags (`{{#foo}}`) 接受一个数据参数，或是没有参数。其它形式的参数会被解析为object specification 或 nested helper：

- Object specification：如果只有关键字参数，例如`{{#with x=1 y=2}}` 或 `{{> prettyBox color=red}}` ，那么关键字参数会被集成到data对象，属性名就是关键字
- Nested Helper：如果有位置参数，
##Template Tag Placement Limitations
和基于纯字符串的模板系统不同，Spacebars 是HTML-aware的，可以自动更新DOM。所以，you can't use a template tag to insert strings of HTML that don't stand on their own, 例如一个单独的开始标签或关闭标签，或是不容易修改的，例如：标签名。

模板标签主要允许存在于三种位置：

- 元素级别
- 属性值
- 在开始标签里，属性键值对


##Double-braced Tags

在元素级别或是属性值位置的双大括号标签通常被解析为一个字符串。如果被解析为其它，则值会被转换成string。当值为null，undefined，或是false时，什么都不会显示。

helper返回的值必须是纯文本，不能是HTMl(换句话说：字符串可以包含`<`，但是不能有`&lt;`)。Spacebars 会执行必要的转义。

### safeString

如果一个在元素级别的双大括号解析为一个对象，由`Spacebars.SafeString("<span>Some HTML</span>")`创建，HTML会被插入到当前的位置。调用SafeString的代码要保证HTML是安全的。

### in Attribute Values

双大括号标签可以作为属性值，或属性值的一部分：

```
<input type="checkbox" class="checky {{moreClasses}}" checked={{isChecked}}>
```

如果属性值是一个双大括号标签，当标签返回null，undefined，或是false时，属性不会出现。否则属性就会出现，即使值为空。
### Dynamic Attributes

双大括号标签可以用在HTML开始标签，用来声明任意的属性：

```
<div {{attrs}}>...</div>

<input type=checkbox {{isChecked}}>
```

标签必须返回一个对象，对象是包含属性名和属性值的字典。为了方便，返回值也可以是字符串或是null。一个空字符串或null会展开成`{}`。一个非空字符串必须是属性名，会被展开成属性等于空值，例如：`"checked"`展开成`{checked:""}`。

总结如下：

|返回值|对应的HTML
|---|:--
"" or null or {}	|
"checked" or {checked: ""}|	checked
{checked: "", 'class': "foo"}	|checked class=foo
"checked class=foo"|	ERROR, string is not an attribute name

可以组合使用动态属性标签：

```
<div id=foo class={{myClass}} {{attrs1}} {{attrs2}}>...</div>

```

动态属性标签产生的属性会从左到右的合并，后出现的属性会覆盖前面的，不会合并。
## Triple-braced Tags
用于在模板中插入原始HTML。

被插入的Html必须包含balanced html tag。例如：你不能插入`</div><div>`来关闭一个已存在的div,然后创建一个新的。

这种模板标签不能用在属性也不能用在开始标签。
## Inclusion Tags
inclusion tag 形式为：`{{> templateName}}` 或 `{{> templateName dataObj}}`。其它形式的参数都是用来构建data对象的语法糖（参见Inclusion and Block Arguments）

inclusion tag 在当前位置插入一个给定模板的实例。如果带参数，参数会成为data context，就如同下面的代码：

```
{{#with dataObj}}
  {{> templateName}}
{{/with}}
```
除了直接指定模板名字外，inclusion tag还可以指定解析为模板对象的path,或是一个返回一个模板对象的函数。

### function returning a template
如果一个inclusion tag 解析为一个函数，那么该函数必须返回一个模板对象或是null。该函数reactively re-run，如果返回值发生变化，模板会被替换。

## Block Tag
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

## If/Unless
当值为`null,undefined,0,"",false`时视为false ，其它值视为true。

```
{{#if something}}
  <p>It's true</p>
{{else}}
  <p>It's false</p>
{{/if}}
```
`#unless` 与 `#if`相反
## With
`#with`标签为它的内容建立了一个新的数据上下文。

```
{{#with employee}}
  <div>Name: {{name}}</div>
  <div>Age: {{age}}</div>
{{/with}}
```

还可以利用object specification，来定义一个对象：

```
{{#with x=1 y=2}}
  {{{getHTMLForPoint this}}}
{{/with}}
```

如果#with的参数值为falsy(和#if的规则相同)，内容不会被渲染。如果提供了else块，就会被渲染。

如果#with的参数值是字符串或其它非对象值，it may be promoted to a JavaScript wrapper object (also known as a boxed value) when passed to helpers, because JavaScript traditionally only allows an object for this. Use String(this) to get an unboxed string value or Number(this) to get an unboxed number value.
## Each

`#each`标签接受一个序列参数，给每一个item渲染内容，将数据上下文设置为当前的item。
```
<ul>
{{#each people}}
  <li>{{name}}</li>
{{/each}}
</ul>
```

`#each`的新变种不改变数据上下文，而是引入了一个新变量，可以用来引用当前item：

```
<ul>
{{#each person in people}}
  <li>{{person.name}}</li>
{{/each}}
</ul>
```


可以提供一个else块，数组长度为0时使用。

在`#each` 块里可以使用`@index` 变量，获取索引。

### reactivity Model for Each
# todo
##Let
`#let`标签用于给指定的表达式创建别名。

```
{{#let name=person.bio.firstName color=generateColor}}
  <div>{{name}} get a {{color}} card!</div>
{{/each}}
```

以这种方式引入的变量优先于模板名，全局help，当前数据上下文的字段和前面用相同名字引入的变量。
## Custom Block Helpers

自定义Block helper很简单，就是声明一个模板，然后用`{{#someTemplate}}`的形式引用，而不是`{{>someTemplate}}`。

当一个模板以Block helper形式调用，它可以使用`{{>Template.contentBlock}}` 和`{{> Template.elseBlock}}`来引入传给他的内容块。

例如：

```
<template name="note">
  <div class="note">
    {{> Template.contentBlock}}
  </div>
</template>
```

用下面方式引用：

```
{{#note}}
  Any content here
{{/note}}
```

下面的例子实现了#unless

```
<template name="unless">
  {{#if this}}
    {{> Template.elseBlock}}
  {{else}}
    {{> Template.contentBlock}}
  {{/if}}
</template>
```

注意#unless的参数变成了模板的数据上下文，通过`this`获取。However, it would not work very well if this data context was visible to Template.contentBlock, which is supplied by the user of unless.

所以，当你引入`{{> Template.contentBlock}}`， Spacebars hides the data context of the calling template, and any data contexts established in the template by #each and #with. They are not visible to the content block, even via ... Put another way, it's as if the {{> Template.contentBlock}} inclusion occurred at the location where {{#unless}} was invoked, as far as the data context stack is concerned.

You can pass an argument to {{> Template.contentBlock}} or {{> Template.elseBlock}} to invoke it with a data context of your choice. You can also use {{#if Template.contentBlock}} to see if the current template was invoked as a block helper rather than an inclusion.
## Comment Tags

```
{{! Start of a section}}
<div class="section">
  ...
</div>
```

注释不会出现在生成的HTML中

注释块：

```
{{!-- This is a block comment.
We can write {{foo}} and it doesn't matter.
{{#with x}}This code is commented out.{{/with}}
--}}
```
注释块可以包含`{{ }}`


## Html Dialect
Spacebars 在运行时会校验你的HTML，如果你违反了基本的HTML语法，导致它无法确定你的HTML结构，spacebars 会抛出一个compile-time 错误。

对不符合规范的标签，spacebars 并不像浏览器那么宽容。
## Top-level Elements in a `.html` file
严格的说，`<template>`标签并不属于Spacebars模板语言。Meteor 中一个`foo.html`模板文件由下面三种元素组成：
- `<template name="myName">`：`<template>`标签包含spacebars 模板，会被编译成`Template.myName`组件
- `<head>`：如果`<head>`标签出现多次（可能是在多个文件中），`<head>`元素的内容会被串联起来。
- `<body>`：会被编译成`Template.body` 组件。如果`<body>`标签出现多次，内容会被串联起来。















