---
layout: post
category : lessons
title: "meteor blaze"
tagline: "Supporting tagline"
tags : [meteor]
---

Blaze 是一个非常强大的库，利用reactive HTML模板来穿件用户界面。使用Blaze，你的APP就无需在写更新逻辑。

模板标签集合了tracker的 transparent reactivity 和miniMongo的 数据库游标，这样DOM就会自动更新。

##细节
Blaze主要包含两部分：
- 一个模板编译器，用来将模板文件编译成javascript对象，
- 一个reactive DOM 引擎，在运行时构建和管理DOM。

##理念
- 平缓的学习曲线
- transparent reactivity。底层，Blaze使用tracker 自动跟踪合适重新计算模板helper。
- 干净的模板
- 可插件

##和其他库对比
- backbone
- ember
- angular polymer
- react

##未来
- 组件
- 表单
- 移动端 动画
- javascript表达式
- 



##什么是reactive 模板
###服务端模板
###jQuery与服务端模板
###客户端模板
Backbone
###reactive 模板
不需要手写javascript来更新DOM

##Blaze是什么？
##模板如何使用Blaze
新一代的模板编译器(例如：Spacebars compiler或Angular‘s template compiler)。
在Meteor中，模板编译器使用Blaze把模板转换成一个函数，这个函数返回一个嵌套的javascript对象数组。
在javascript代码中，每个模板被拆分成regions of reactively-updating DOM components (Blaze Views) 和 nonreactive DOM nodes (HTMLJS)。运行时，Blaze执行那个函数，然后渲染它的输出，将Blaze views 和HTMLJS对象转换为DOM node。

将模板转换成一个标准的javascript对象来表示，使得使用多种模板语言变得很容易。更棒的是，它使Blaze可以细粒度更新。Reactive values ，内置表达式（例如：{#if}）,甚至模板本身都由reactive-updating Blaze view 来表示。当reactive value或内置表达式发生变化时，Blaze只更新关联到那个值的DOM节点，其它地方不变。

###编译时：从Spacebars 到javascript
首先，Meteor创建了一个全局Template对象，是键值对形式，用来通过name查找模板信息。

接下来，模板编译器使用Blaze把模板标签转换成一个函数。编译器生成的javascript代码会把模板的名字和它的渲染函数保存到全局的Template对象中。

渲染函数是模板结构的javascript表示。它返回一个嵌套的HTMLJS 和Blaze view

###运行时：从javascript到html

- 第一次运行
Blaze用在全局Template 对象中找到的名字和渲染函数给模板创建一个Blaze view，Blaze还会创建view 对应enclosed template 或是内置的Block helper。

接下来，Blaze递归的渲染模板的view，把渲染函数返回的HTMLJS转换成DOMRange（连续的DOM节点块，对应view的内容）。模板view里的子view也会被渲染到DOMRange.如果一个view的父view被插入到DOM里，那么该view的DOMRange也会被插入到DOM。也就是说，如果模板A在模板B里面，模板B的view插入到DOM之后，模板A的view也被插入到DOM中

UI.body 是另外一个模板对象。运行时，Blaze把body模板渲染成DOMRange,插入到document.body.

- Reactive更新
如果HTMLJS里的一个reactive 值发生变化，view会被重新渲染。也就是说对应该view的DOMRange会从DOM中移除，用新的替换。


##Blaze views

Blaze用view来创建reactively-updating region of DOM。最常见的两种view对应模板和内置的helper。模板的view只会被渲染到DOM一次，而内置helper 的view可能会被渲染多次。

###功能
- 生命周期回调
- 父指针
- 渲染方法
- DOMRange

###生命周期
- Construction ：通过调用 new Blaze.view 构建一个view。你可以attach 事件钩子，例如onRendered 和 onCreated。注意：此时还没有created
- Creation ：当在view上调用Blaze.render() 的时候，将渲染函数转换成DOMRange，然后插入到DOM中。 当Blaze.render()运行的时候，onCreated 回调函数被调用。
- Rendering ：当View的DOMRange插入到DOM中(Blaze.render()的结果)之后 and teh next Deps flush cycle occurs，onRendered回调函数被调用
- Destruction ： 无论何时view 被destroy或是从DOM中移除，onDestroyed 被调用。可以通过view.destroyView() 或是移除HTML元素的方式














