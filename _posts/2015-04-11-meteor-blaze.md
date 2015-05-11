---
layout: post
category : lessons
title: "meteor blaze 手册"
tagline: "Supporting tagline"
tags : [meteor]
---
##概述
Blaze是一个前端库，为Meteor提供了reactive 模板。非常简单，高效，也可以在Meteor之外使用。

下面介绍Blaze背后的理念并展示如何使用Blaze实现模板系统。

##什么是reactive 模板
###服务端模板

直到今天，web应用主要还是依赖服务端执行逻辑和渲染模板。当用户导航到一个特定URL时，浏览器发送一个HTTP请求到服务器，服务器把对应的模板渲染成HTML字符串，返回给浏览器。

###jQuery与服务端模板

####使用DOM API

服务端模板适合那些内容不经常变化的应用。最终，那些发展为动态应用的，它们需要更新来响应用户的操作，而又不能刷新页面。这些应用依赖于服务端渲染初始页面，然后使用jQuery或是原生javascript来局部更新DOM。

jQuery对简单地DOM操作是非常高效的。但是对于复杂的数据操作就力不从心了。如果在考虑到Ajax与服务器通信，保持数据的同步就更加复杂。

因为纯jQuery使开发者被迫用DOM作为他们的数据存储。开发者必须手动的选择需要更新的DOM节点，如果更新很大，那么就很复杂。


####使用Data Models

随着web应用越来越复杂，data model出现了，用来转移从DOM跟踪数据的压力。一个data model 可能是一个来自服务端的JSON对象，或是一系列对象，或是保存着当前状态的本地变量。开发者越来越多的采用MVC，model是数据的表示，而view的职责就是简单地反映model。

###客户端模板
客户端模板引擎，例如Backbone等，是另一种可以动态更新的方式。它们把HTML拆成独立的view，不用写jQuery来生成或是更新HTML，你要做的就是re-run view。但是代价很高，当HTML输出不再有效时，模板引擎需要替换整个DOM 区域。这会导致性能问题和元素丢失状态。

###reactive 模板

reactive 模板使编写高效、易懂的应用代码成为可能。模板指定可能发生变化的DOM段，然后一个强大的前端框架(例如：Meteor的Blaze，React或是Angular)自动的更新DOM。无需再手写javascript来更新DOM。

##Blaze是什么？

Blaze是一个前端框架用来构建reactive模板，它有几个明显的特点：

- Declarative templates: 声明what,而不是how。
- fine-grained updates
- automatically managed dependencies:无需声明依赖。模板里的reactive变量通过tracker跟踪。
- interoperability 互操作性
- 容易理解
- syntax-agnostic


##模板如何使用Blaze

![http://manual.meteor.com/blaze-9.png](http://manual.meteor.com/blaze-9.png)

大多数模板引擎都是执行字符串替换。如果模板只是渲染一次，那么字符串替换非常高效。但是它无法应对动态更新。


新一代的模板编译器(例如：Spacebars compiler或Angular‘s template compiler)理解模板的结构。它把模板分割成reactive 和nonreactive 块。每个块对应一部分DOM。当数据变化时，只重新渲染reactive 块。

![http://manual.meteor.com/blaze-jsbox.png](http://manual.meteor.com/blaze-jsbox.png)

在Meteor中，模板编译器使用Blaze把模板转换成一个函数，这个函数返回一个嵌套的javascript对象数组。
在javascript代码中，每个模板被拆分成regions of reactively-updating DOM components (Blaze Views) 和 nonreactive DOM nodes (HTMLJS)。运行时，Blaze执行那个函数，然后渲染它的输出，将Blaze views 和HTMLJS对象转换为DOM node。

将模板转换成一个标准的javascript对象来表示，使得使用多种模板语言变得很容易。更棒的是，它使Blaze可以细粒度更新。Reactive values ，内置表达式（例如：{#if}）,甚至模板本身都由reactive-updating Blaze view 来表示。当reactive value或内置表达式发生变化时，Blaze只更新关联到那个值的DOM节点，其它地方不变。

上图红框和绿框代表不同的Blaze view。当把模板从spacebars 转换为javascript时，Blaze创建一个view 对应{{currentUser}} ,这个view包含一个reactive 值{{view.lookup("currentUser")}} ,随着登录用户不同而改变。

接下来是更详细的介绍：

###编译时：从Spacebars 到javascript
首先，Meteor创建了一个全局Template对象，是键值对形式，用来保存模板信息，可以用name查找。

接下来，模板编译器使用Blaze把模板标签转换成一个函数。编译器生成的javascript代码会把模板的名字和它的渲染函数保存到全局的Template对象中。

渲染函数是模板结构的javascript表示。它返回一个嵌套的HTMLJS 和Blaze view

###运行时：从javascript到html

- 第一次运行
Blaze用在全局Template 对象中找到的名字和渲染函数给模板创建一个Blaze view，Blaze还会创建view 对应嵌套的模板或是内置的Block helper。

接下来，Blaze递归的渲染模板的view，把渲染函数返回的HTMLJS转换成DOMRange（连续的DOM节点块，对应view的内容）。模板view里的子view也会被渲染到DOMRange.如果一个view的父view被插入到DOM里，那么该view的DOMRange也会被插入到DOM。也就是说，如果模板A在模板B里面，模板B的view插入到DOM之后，模板A的view也被插入到DOM中

UI.body 是另外一个模板对象。运行时，Blaze把body模板渲染成DOMRange,插入到document.body.

- Reactive更新
如果HTMLJS里的一个reactive 值发生变化，view会被重新渲染。也就是说对应该view的DOMRange会从DOM中移除，用新的替换。


##Blaze views

Blaze用view来创建reactively-updating region of DOM。最常见的两种view对应模板和内置的helper。模板的view只会被渲染到DOM一次，而内置Block helper 的view可能会被渲染多次。

###功能

- 生命周期回调
- 父指针
- 渲染方法
- DOMRange:当一个view渲染到DOM中，它在DOM中的位置和边界由一个DOMRange对象来跟踪。

###生命周期

![http://manual.meteor.com/blaze-view-lifecycle.png](http://manual.meteor.com/blaze-view-lifecycle.png)

- Construction ：通过调用 new Blaze.view 构建一个view。你可以attach 事件钩子，例如onRendered 和 onCreated。注意：此时还没有created
- Creation ：当在view上调用Blaze.render() 的时候，将渲染函数转换成DOMRange，然后插入到DOM中。 当Blaze.render()运行的时候，onCreated 回调函数被调用。
- Rendering ：当View的DOMRange插入到DOM中(Blaze.render()的结果)之后 and the next Deps flush cycle occurs，onRendered回调函数被调用
- Destruction ： 无论何时view 被destroy或是从DOM中移除，onDestroyed 被调用。可以通过view.destroyView() 或是移除HTML元素的方式

###代表模板的view
如上所述，在运行时，Blaze为每一个模板创建一个view对象。这个对象包含模板的名字和渲染函数(代表模板结构的javascript函数)。然后在这个view对象上调用Blaze.render()，把view 返回的HTMLJS对象转换为连续的DOM节点。

模板的生命周期直接对应view的生命周期。如果开发者定义了模板的onCreated回调，就会在对应view的onCreated回调中调用。

###代表内置Block helper的view
Blaze Block helper 有Blaze view和tracker 实现。例如：Blaze.view

```
Blaze.With = function (data, contentFunc) {
  var view = Blaze.View('with', contentFunc);
  view.dataVar = new Blaze.ReactiveVar;

  view.onCreated(function () {
    if (typeof data === 'function') {
      // `data` is a reactive function
      view.autorun(function () {
        view.dataVar.set(data());
      }, view.parentView);
    } else {
      view.dataVar.set(data);
    }
  });

  return view;
};
```

上面的代码初始化了一个Blaze.View 和一个 dataVar，一个用来保存当前数据上下文的reactive变量。当view created时，会触发它的onCreated回调函数，用view.dataVar.set() 设置数据上下文。

设置数据上下文的代码包裹在一个view.autorun中。当view.autorun中的reactive 变量变化时，autorun重新运行，就会重新设置数据上下文。

##Blaze，jQuery，和reactive 属性
###Blaze如何处理属性
参考：[http://manual.meteor.com/#blaze](http://manual.meteor.com/#blaze)










