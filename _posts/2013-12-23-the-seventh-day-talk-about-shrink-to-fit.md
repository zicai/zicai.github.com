---
layout: post
title: "第七天，谈谈【shrink-to-fit】"
description: ""
category: 
tags: []
---

## 什么是 shrink-to-fit##
shrink-to-fit指的是块级元素只占据内容所需要的宽度。[Shrink-To-Fit](http://www.w3.org/TR/CSS21/visudet.html#shrink-to-fit-float)，字面意思就是**收缩包围**。
## 为什么需要shrink-to-fit ##
我们都知道在默认情况下，块级元素（更确切的说是`display`属性值为`block`或`list-item`的元素）在横向上会占据它所能占据的最大宽度。但是，如果我只想让块元素和它所包含的内容一样宽，该怎么做呢？最容易想到的方法是：设置`width`。可是，如果内容是动态的，宽度不固定的呢？这时，我们就可以利用shrink-to-fit了。

##如何使块元素shrink-to-fit##
 下面我们将介绍五种常见的使块元素shrink-to-fit的方式，并顺带介绍一个很常见的需求：如何使shrink-to-fit的元素相对于父元素居中显示。

 - 方式一：通过`float`属性

 示例代码一：
 <iframe width="100%" height="300" src="http://jsfiddle.net/zicai/9nJDc/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>
 这应该是使用最普遍的一种方式，因为主流浏览器都支持，并且浮动带来的副作用很小（一般通过[清除浮动](http://zicai.github.io/2013/12/15/the-third-day-talk-about-clearfix-part-one/)即可消除副作用）。

 但如果你想让shrink-to-fit的元素居中显示，可能就要换一种方式了，比如下面这种。

 - 方式二：通过`display:inline`

 示例代码二：
 <iframe width="100%" height="300" src="http://jsfiddle.net/zicai/9nJDc/1/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>
 把`shrink`元素设置成`display:inline`就可以使它和内容一样宽。而通过`container`的`text-align:center`即可使其居中显示。但是`display:inline`带来的副作用也有很多，例如：

  - 将无法设置`shrink`元素的`width`和`height`
  - 将无法设置`shrink`元素的垂直`padding`
  - 如果`shrink`元素包含多行内容，并且`shrink`元素有边框或背景时，效果会一团糟。如下：
  
 <iframe width="100%" height="300" src="http://jsfiddle.net/zicai/9nJDc/2/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

 - 方式三：通过`display:inline-block`

  示例代码三：
 <iframe width="100%" height="300" src="http://jsfiddle.net/zicai/9nJDc/3/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

 `display:inline-block`在现代浏览器中即可使元素shrink-to-fit，为了在IE6、7中达到同样的效果，需要用`*display: inline;*zoom: 1;`hack一下。详见[上一篇谈谈【inline-block】](http://zicai.github.io/2013/12/20/the-sixth-day-talk-about-inline-block/)

 用这种方式shrink-to-fit的元素，通过设置`container`的`text-align:center`即可使其居中显示。

 - 方式四：通过`position:absolute`

 示例代码四：
 <iframe width="100%" height="300" src="http://jsfiddle.net/zicai/9nJDc/4/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>
 这应该是用的最少的一种方式，因为`position:absolute`会改变元素的定位方式，使元素脱离普通文档流。(由于固定定位`position:fixed`是绝对定位`position:absolute`的一种，所以`position:fixed`同样可以使元素shrink-to-fit，这里不再展开叙述)

 - 方式五：通过`display:table`

 示例代码五：
 <iframe width="100%" height="300" src="http://jsfiddle.net/zicai/9nJDc/6/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

 `display:table`即可使元素shrink-to-fit，再加上`margin: 0 auto`即可使其居中显示。可惜的是IE6、7不支持这种方式。

除了上面这五种常见的方式，还有一些不常用的，例如：`display: table-cell`、 `display: inline-table`。

在IE中也有一些和shrink-to-fit相关的bug，例如：[当一个应该收缩包围的元素中包含一个拥有“layout”的块级元素时](http://www.html-js.com/article/Where-is-Bug-The-second-set-IE-shrinkwrap-Bug)，会出现一些意想不到的现象，相关bug会在专栏[Bug 去哪儿？](http://www.html-js.com/article/column/81)中出现，欢迎前去订阅。本文的主要目的也是为说明IE 中shrink-to-fit bug 做准备。

参考文章：

 - [http://haslayout.net/css-tuts/CSS-Shrink-Wrap](http://haslayout.net/css-tuts/CSS-Shrink-Wrap)
 - [http://www.brunildo.org/test/shrink-to-fit.html](http://www.brunildo.org/test/shrink-to-fit.html)(需翻墙)



题外话：

分享一段话：来自熊培云的[《自由在高处》](http://book.douban.com/subject/5401989/)

> 在一个广场上，人挤人，你不知道方向在哪里，但如果你站得高一点，看得远一点，就知道周遭的种种拥挤对你来说其实毫无意义。

快过年了，提前祝炖友们过年好，O(∩_∩)O

  [1]: http://jsfiddle.net/zicai/9nJDc/3/embedded/