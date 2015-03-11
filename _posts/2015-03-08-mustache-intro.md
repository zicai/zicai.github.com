---
layout: post
category : lessons
title: "Mustache 基础"
tagline: "Supporting tagline"
tags : [Mustache]
---

##概要
一个典型的Mustache模板：

```   
Hello {{name}}
You have just won {{value}} dollars!
{{#in_ca}}
Well, {{taxed_value}} dollars, after taxes.
{{/in_ca}}
```

如果给定下面的hash

    {
      "name": "Chris",
      "value": 10000,
      "taxed_value": 10000 - (10000 * 0.4),
      "in_ca": true
    }
    
会输出下面的内容：

    Hello Chris
    You have just won 10000 dollars!
    Well, 6000.0 dollars, after taxes.


##说明
Mustache可以用在HTML，配置文件，源代码--等任何文件中。它的工作方法是：用以hash或者object形式提供的值展开模板里的tags。

说它logic-less是因为它没有if else语句，也没有for循环。它只有tags。tag可以被一个值替换，也可以是一系列值。

##Tag 类型
Tag用双大括号标明。{{person}}是一个tag，{{#person}}也是一个tag。下面的例子中，我们把 person 叫做`key` 或者 tag key。

tag类型包括:

- Variables
- Sections
- Inverted Section
- Comments
- Partials
- Set Delimiter

###Variables
最基本的tag就是variable。模板里的一个 {{name}} tag会尝试在当前context中寻找`name` key。如果没找到，则递归的从父级context中查找。如果一直到顶层context，都没找到`name` key。那么就不会渲染。

默认所有的variable都会进行HTML escape。如果你想返回unescaped html，使用{{{name}}}

你还可以使用`&`来unescaped 一个variable：{{& name}}。这在改变定界符时很有用。

默认情况下，一个variable “miss”会返回一个空字符串。通常这可以在你的Mustache库中进行配置。例如：Ruby版的Mustache支持在这种情况下抛出一个错误。

模板：

    * {{name}}
    * {{age}}
    * {{company}}
    * {{{company}}}
    
hash:

    {
      "name": "Chris",
      "company": "<b>GitHub</b>"
    }

输出：

    * Chris
    *
    * &lt;b&gt;GitHub&lt;/b&gt;
    * <b>GitHub</b>

###Sections
sections会根据当前context中key的值渲染文本块一次或多次。

一个section以`#`开始，`/`结束。例如：{{#person}}开始一个“person” section ，而{{/person}}结束它。我们把开始tag 和结束tag之间的文本叫做section's "block"(后面叫做文本块)。

section的行为取决于key的值。key的值有下面几种情况：

- False Values or Empty Lists
- Non-Empty Lists
- Lambdas
- Non-False Values

**False Values or Empty Lists** 
如果person key存在，值为false或者是一个empty list,`#`和`/`之间的HTML不会被显示出来。

模板：

    Shown.
    {{#person}}
      Never shown!
    {{/person}}
    
Hash:

    {
      "person": false
    }
    
输出：

    Shown.

**Non-Empty Lists**
如果person key存在，并且值为non-false。`#`和`/`之间的HTML会被渲染一次或多次。

如果值为非空列表，列表里的每一个item都会渲染一次文本块。
每次iteration，文本块的context会被设置为当前的item。我们可以用这种方式循环列表。

模板：

    {{#repo}}
      <b>{{name}}</b>
    {{/repo}}

hash:

    {
      "repo": [
        { "name": "resque" },
        { "name": "hub" },
        { "name": "rip" }
      ]
    }
    
输出：

    <b>resque</b>
    <b>hub</b>
    <b>rip</b>
    
**Lambdas**
如果值为callable object，例如：函数或者lambda。object会被调用，同时文本块会作为参数传进去。文本块以literal block形式传进去，还未渲染。即tag 还未展开，lambda需要自己做这件事。可以据此来实现filters或者caching

模板：

    {{#wrapped}}
      {{name}} is awesome.
    {{/wrapped}}

hash:

    {
      "name": "Willy",
      "wrapped": function() {
        return function(text, render) {
          return "<b>" + render(text) + "</b>"
        }
      }
    }

输出：

    <b>Willy is awesome.</b>

**Non-False Values**
如果值为non-false，但不是list，它会用作context渲染一次文本块。

模板：

    {{#person?}}
      Hi {{name}}!
    {{/person?}}

hash:

    {
      "person?": { "name": "Jon" }
    }

输出：

    Hi Jon!

###Inverted Section
一个inverted section 以`^`开始，以`/`结束。sections用来根据key的值渲染文本块一次或者多次，inverted sections根据key的inverse value渲染文本一次。也就是说，如果key不存在或者是false，或者是空list，才会被渲染。

模板：

    {{#repo}}
      <b>{{name}}</b>
    {{/repo}}
    {{^repo}}
      No repos :(
    {{/repo}}

hash:

    {
      "repo": []
    }

输出：

    No repos :(

###Comments
注释以感叹号开始，并且不会被渲染。例如：

    <h1>Today{{! ignore me }}.</h1>

会出输出：

    <h1>Today.</h1>

注释可以包含新行。
###Partials
partials以 `>` 开始，例如：{{> box}}。

partials在运行时渲染(和编译时相对)，所以递归的partials是有可能的。只是要避免无限循环。

partials会继承calling context。在一个[ERB](http://en.wikipedia.org/wiki/ERuby)文件中，你可能需要这样：

    <%= partial :next_more, :start => start, :size => size %>

但是用Mustache，你只需：

    {{> next_more}}

为什么？因为next.mustache文件会从calling context继承size 和start method。


###Set Delimiter
set delimiter tag以`=`开始，将定界符从`{{` 和`}}`修改为自定义的字符串。

例如：

    * {{default_tags}}
    {{=<% %>=}}
    * <% erb_style_tags %>
    <%={{ }}=%>
    * {{ default_tags_again }}

自定义的定界符不能包含等号和空格