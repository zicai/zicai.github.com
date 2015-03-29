---
layout: post
category : lessons
title: "Mustache.js 基础"
tagline: "Supporting tagline"
tags : [Mustache]
---

mustache.js 是mustache模板系统的javascript实现。

你可以在任何可以使用javascript的地方使用mustache.js。例如：浏览器，服务器端(node，CouchDB views)。

mustache.js支持CommonJS module API，也支持[Asynchronous Module Definition API](https://github.com/amdjs/amdjs-api/wiki/AMD)

##使用

使用mustache.js简单例子:

```js
var view = {
  title: "Joe",
  calc: function () {
    return 2 + 4;
  }
};

var output = Mustache.render("{{title}} spends {{calc}}", view);
```

`Mustache.render`函数接受两个参数：

1）mustache模板 

2）一个`view`对象，它包含需要渲染到模板中的数据和代码

## 模板

mustache模板是包含若干mustache tag的字符串。

有很多种加载模板的方式，其中两种是：


#### Include Templates

直接写在HTML文件中，下例中用到了`jQuery`:

```html
<html>
<body onload="loadUser">
<div id="target">Loading...</div>
<script id="template" type="x-tmpl-mustache">
Hello {{ name }}!
</script>
</body>
</html>
```

```js
function loadUser() {
  var template = $('#template').html();
  Mustache.parse(template);   // optional, speeds up future uses
  var rendered = Mustache.render(template, {name: "Luke"});
  $('#target').html(rendered);
}
```

#### Load External Templates

如果你的模板存在于独立文件中，你可以异步加载它们，加载完成后再渲染，同样用到了`jQuery`:

```js
function loadUser() {
  $.get('template.mst', function(template) {
    var rendered = Mustache.render(template, {name: "Luke"});
    $('#target').html(rendered);
  });
}
```
mustache tag 有下面几种，详细内容参见[Mustache 基础](https://github.com/zicai/zicai.github.com/blob/master/_posts/2015-03-08-mustache-intro.md)。后面我们只谈mustache.js 特殊的地方。

### Variables

可以使用JavaScript's dot notation 获取view中对象的属性

View:

```json
{
  "name": {
    "first": "Michael",
    "last": "Jackson"
  },
  "age": "RIP"
}
```

Template:

```html
* {{name.first}} {{name.last}}
* {{age}}
```

Output:

```html
* Michael Jackson
* RIP
```

### Sections


#### False Values or Empty Lists

如果`person`键不存在，或者存在但值为`null`, `undefined`, `false`, `0`, or `NaN`,或者是一个空字符串或者空列表，则不会渲染。

View:

```json
{
  "person": false
}
```

Template:

```html
Shown.
{{#person}}
Never shown!
{{/person}}
```

Output:

```html
Shown.
```

#### Non-Empty Lists

如果`person`键存在，且值为非空列表，则渲染一次或多次。


View:

```json
{
  "stooges": [
    { "name": "Moe" },
    { "name": "Larry" },
    { "name": "Curly" }
  ]
}
```

Template:

```html
{{#stooges}}
<b>{{name}}</b>
{{/stooges}}
```

Output:

```html
<b>Moe</b>
<b>Larry</b>
<b>Curly</b>
```

在循环字符串数组时，`.`可以用来引用当前的item

View:

```json
{
  "musketeers": ["Athos", "Aramis", "Porthos", "D'Artagnan"]
}
```

Template:

```html
{{#musketeers}}
* {{.}}
{{/musketeers}}
```

Output:

```html
* Athos
* Aramis
* Porthos
* D'Artagnan
```

如果一个section variable是一个函数，它会在当前item的context中调用。(注意和下面的函数块相区别，类似于`each` 和 `with`) 

View:

```js
{
  "beatles": [
    { "firstName": "John", "lastName": "Lennon" },
    { "firstName": "Paul", "lastName": "McCartney" },
    { "firstName": "George", "lastName": "Harrison" },
    { "firstName": "Ringo", "lastName": "Starr" }
  ],
  "name": function () {
    return this.firstName + " " + this.lastName;
  }
}
```

Template:

```html
{{#beatles}}
* {{name}}
{{/beatles}}
```

Output:

```html
* John Lennon
* Paul McCartney
* George Harrison
* Ringo Starr
```

#### Functions
It is called in the context of the current view object.

View:

```js
{
  "name": "Tater",
  "bold": function () {
    return function (text, render) {
      return "<b>" + render(text) + "</b>";
    }
  }
}
```

Template:

```html
{{#bold}}Hi {{name}}.{{/bold}}
```

Output:

```html
<b>Hi Tater.</b>
```

### Inverted Sections

只有当值为 `null`, `undefined`, `false`, *falsy* or an empty list 时才会渲染。

View:

```json
{
  "repos": []
}
```

Template:

```html
{{#repos}}<b>{{name}}</b>{{/repos}}
{{^repos}}No repos :({{/repos}}
```

Output:

```html
No repos :(
```

### Comments

```html
<h1>Today{{! ignore me }}.</h1>
```

Will render as follows:

```html
<h1>Today.</h1>
```



### Partials

在mustache.js中，一个partials对象可以作为第三个参数传给`Mustache.render`。partial对象的key必须为partial的name,value是partial text。


```js
Mustache.render(template, view, {
  user: userTemplate
});
```

### Set Delimiter


```
* {{ default_tags }}
{{=<% %>=}}
* <% erb_style_tags %>
<%={{ }}=%>
* {{ default_tags_again }}
```



## Pre-parsing and Caching Templates

默认情况下，mustache.js 第一次解析一个模板之后，它会缓存整个parsed token tree。下次它遇到相同的template时，就会跳过解析的步骤，更快的渲染模板。如果你愿意，你可以提前使用`mustache.parse`来做解析。

```js
Mustache.parse(template);

// Then, sometime later.
Mustache.render(template, view);
```

## Plugins for JavaScript Libraries

mustache.js may be built specifically for several different client libraries, including the following:

  - [jQuery](http://jquery.com/)
  - [MooTools](http://mootools.net/)
  - [Dojo](http://www.dojotoolkit.org/)
  - [YUI](http://developer.yahoo.com/yui/)
  - [qooxdoo](http://qooxdoo.org/)

These may be built using [Rake](http://rake.rubyforge.org/) and one of the following commands:

    $ rake jquery
    $ rake mootools
    $ rake dojo
    $ rake yui3
    $ rake qooxdoo

## Command line tool

mustache.js 自带一个基于node的命令行工具，它可以作为全局工具安装。

```bash
$ npm install -g mustache
$ mustache dataView.json myTemplate.mustache > output.html

# also supports stdin
$ cat dataView.json | mustache - myTemplate.mustache > output.html
```

or as a package.json `devDependency` in a build process maybe?

```bash
$ npm install mustache --save-dev
```
```json
{
  "scripts": {
    "build": "mustache dataView.json myTemplate.mustache > public/output.html"
  }
}
```
```bash
$ npm run build
```


命令行工具基本上是`Mustache.render`的一个包装，所以上面提到的功能都具备。



