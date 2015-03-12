---
layout: post
category : lessons
title: "Handlebars.js 基础"
tagline: "Supporting tagline"
tags : []
---

Handlebars.js 是Mustache 模板语言的一个扩展。Handlebar.js 和Mustache都是logicless模板语言，目的是保持视图和逻辑分开。

[Handlebars.js 文档](http://www.handlebarsjs.com)

##使用

大体上，Handlerbars.js的语法是Mustache的超集。基本语法可以查看[Mustache manpage](https://github.com/zicai/zicai.github.com/blob/master/_posts/2015-03-08-mustache-intro.md)


拿到模板以后，用`Handlebars.compile`方法将模板编译成一个函数。生成的这个函数接收一个context参数，它会被用来渲染模板。

```js
var source = "<p>Hello, my name is {{name}}. I am from {{hometown}}. I have " +
             "{{kids.length}} kids:</p>" +
             "<ul>{{#kids}}<li>{{name}} is {{age}}</li>{{/kids}}</ul>";
var template = Handlebars.compile(source);

var data = { "name": "Alan", "hometown": "Somewhere, TX",
             "kids": [{"name": "Jimmy", "age": "12"}, {"name": "Sally", "age": "4"}]};
var result = template(data);

// Would render:
// <p>Hello, my name is Alan. I am from Somewhere, TX. I have 2 kids:</p>
// <ul>
//   <li>Jimmy is 12</li>
//   <li>Sally is 4</li>
// </ul>
```


##注册 Helpers

你可以注册helpers，Handlebars会在渲染模板过程中使用它们。例如：

```js
Handlebars.registerHelper('link_to', function() {
  return new Handlebars.SafeString("<a href='" + Handlebars.Utils.escapeExpression(this.url) + "'>" + Handlebars.Utils.escapeExpression(this.body) + "</a>");
});

var context = { posts: [{url: "/hello-world", body: "Hello World!"}] };
var source = "<ul>{{#posts}}<li>{{link_to}}</li>{{/posts}}</ul>"

var template = Handlebars.compile(source);
template(context);

// Would render:
//
// <ul>
//   <li><a href='/hello-world'>Hello World!</a></li>
// </ul>
```


Helpers 优先级高于context中定义的fields,要想获取被helper掩盖的field，可以使用path reference。在上例中，如果在`context`对象中有一个叫做`link_to`的field,可以用下面方式引用它：

```
{{./link_to}}
```

##转义

默认情况下，`{{expression}}`语法会转义它的内容。避免遭到服务器传来的恶意JSON数据导致的XSS攻击。

要想明确不转义内容，使用`{{{}}}`。

Handlebars.js 与 Mustache 的不同
----------------------------------------------

为了更加容易的编写模板，Handlebars.js添加了一些额外的功能，也修改了一些partial的工作方式。

### Paths

Handlebars.js支持一个扩展的表达式语法，我们称为paths。paths 由普通的表达式和`.`组成。path使你不仅可以显示当前context的数据，还可以显示当前context的祖先和后代的context的数据。

要显示后代context的数据，使用`.`字符。假设数据结构如下：

```js
var data = {"person": { "name": "Alan" }, "company": {"name": "Rad, Inc." } };
```

在top-level context可以用下面的表达式显示person's name：

```
{{person.name}}
```

可以使用`../`后退。假设你已经进入了person对象，仍然可以通过表达式`{{../company.name}}`来显示company's name，所以：

```
{{#with person}}{{name}} - {{../company.name}}{{/with}}
```

would render:

```
Alan - Rad, Inc.
```

### Strings

在调用helper时，可以传递paths或者strings 作为参数，例如：

```js
Handlebars.registerHelper('link_to', function(title, options) {
  return "<a href='/posts" + this.url + "'>" + title + "!</a>"
});

var context = { posts: [{url: "/hello-world", body: "Hello World!"}] };
var source = '<ul>{{#posts}}<li>{{{link_to "Post"}}}</li>{{/posts}}</ul>'

var template = Handlebars.compile(source);
template(context);

// Would render:
//
// <ul>
//   <li><a href='/posts/hello-world'>Post!</a></li>
// </ul>
```


### Block Helpers



Handlebars.js 增加了定义block helpers的能力。block helpers 是可以在模板任意地方调用的函数。例如：

```js
var source = "<ul>{{#people}}<li>{{#link}}{{name}}{{/link}}</li>{{/people}}</ul>";
Handlebars.registerHelper('link', function(options) {
  return '<a href="/people/' + this.id + '">' + options.fn(this) + '</a>';
});
var template = Handlebars.compile(source);

var data = { "people": [
    { "name": "Alan", "id": 1 },
    { "name": "Yehuda", "id": 2 }
  ]};
template(data);

// Should render:
// <ul>
//   <li><a href="/people/1">Alan</a></li>
//   <li><a href="/people/2">Yehuda</a></li>
// </ul>
```

Whenever the block helper is called it is given one or more parameters,
any arguments that are passed into the helper in the call, and an `options`
object containing the `fn` function which executes the block's child.
The block's current context may be accessed through `this`.

Block helpers have the same syntax as mustache sections but should not be
confused with one another. Sections are akin to an implicit `each` or
`with` statement depending on the input data and helpers are explicit
pieces of code that are free to implement whatever behavior they like.
The [mustache spec](http://mustache.github.io/mustache.5.html)
defines the exact behavior of sections. In the case of name conflicts,
helpers are given priority.

### Partials

你可以注册额外的模板作为partial，当Handlebars遇到`{{> partialName}}`时就会使用。Partials既可以是一个模板字符串，也可以是一个编译后的模板函数。例如：

```js
var source = "<ul>{{#people}}<li>{{> link}}</li>{{/people}}</ul>";

Handlebars.registerPartial('link', '<a href="/people/{{id}}">{{name}}</a>')
var template = Handlebars.compile(source);

var data = { "people": [
    { "name": "Alan", "id": 1 },
    { "name": "Yehuda", "id": 2 }
  ]};

template(data);

// Should render:
// <ul>
//   <li><a href="/people/1">Alan</a></li>
//   <li><a href="/people/2">Yehuda</a></li>
// </ul>
```

### Comments

可以用下面的语法在模板中添加注释：

```js
{{! This is a comment }}
```

如果你想让注释出现在最终输出，可以使用html 注释。

```html
<div>
    {{! This comment will not end up in the output }}
    <!-- This comment will show up in the output -->
</div>
```


### Compatibility

有一小部分Mustache行为，Handlebars没有实现

- 默认情况下，Handlebars没有遵循Mustache的递归查询。要想启用这个功能，必须设置compile time `compat`。你要清楚，开启这个flag会导致性能下降。
- 不支持Mustache-style lambdas。Handlebars提供了自己的lambda解决方案：以helpers的形式。
- 不支持变更定界符


预编译模板
----------------------

为了速度更快，Handlebars允许预编译模板，然后以javascript代码形式引入。

### 安装

预编译脚本可以通过npm安装：

    npm install -g handlebars

### 使用

<pre>
Precompile handlebar templates.
Usage: handlebars template...

Options:
  -a, --amd            Create an AMD format function (allows loading with RequireJS)          [boolean]
  -f, --output         Output File                                                            [string]
  -k, --known          Known helpers                                                          [string]
  -o, --knownOnly      Known helpers only                                                     [boolean]
  -m, --min            Minimize output                                                        [boolean]
  -s, --simple         Output template function only.                                         [boolean]
  -r, --root           Template root. Base value that will be stripped from template names.   [string]
  -c, --commonjs       Exports CommonJS style, path to Handlebars module                      [string]
  -h, --handlebarPath  Path to handlebar.js (only valid for amd-style)                        [string]
  -n, --namespace      Template namespace                                                     [string]
  -p, --partial        Compiling a partial template                                           [boolean]
  -d, --data           Include data when compiling                                            [boolean]
  -e, --extension      Template extension.                                                    [string]
  -b, --bom            Removes the BOM (Byte Order Mark) from the beginning of the templates. [boolean]
</pre>

If using the precompiler's normal mode, the resulting templates will be
stored to the `Handlebars.templates` object using the relative template
name sans the extension. These templates may be executed in the same
manner as templates.

If using the simple mode the precompiler will generate a single
javascript method. To execute this method it must be passed to
the `Handlebars.template` method and the resulting object may be used as normal.

### 优化

- Rather than using the full _handlebars.js_ library, implementations that
  do not need to compile templates at runtime may include _handlebars.runtime.js_
  whose min+gzip size is approximately 1k.
- If a helper is known to exist in the target environment they may be defined
  using the `--known name` argument may be used to optimize accesses to these
  helpers for size and speed.
- When all helpers are known in advance the `--knownOnly` argument may be used
  to optimize all block helper references.
- Implementations that do not use `@data` variables can improve performance of
  iteration centric templates by specifying `{data: false}` in the compiler options.

Supported Environments
----------------------

Handlebars has been designed to work in any ECMAScript 3 environment. This includes

- Node.js
- Chrome
- Firefox
- Safari 5+
- Opera 11+
- IE 6+

Older versions and other runtimes are likely to work but have not been formally
tested. The compiler requires `JSON.stringify` to be implemented natively or via a polyfill. If using the precompiler this is not necessary.

[![Selenium Test Status](https://saucelabs.com/browser-matrix/handlebars.svg)](https://saucelabs.com/u/handlebars)

External Resources
------------------

* [Gist about Synchronous and asynchronous loading of external handlebars templates](https://gist.github.com/2287070)


