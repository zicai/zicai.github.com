---
layout: post
title: "第四天，谈谈【清除浮动 (下)】"
description: ""
category: 
tags: []
---

在上一篇：[清除浮动 (上)](http://www.html-js.com/article/1691)中我们提到了两种清除浮动的方式：

 - 方法一：直接在HTML中插入带有 `clear` 属性的标签
 - 方法二：使用 `:after` 伪类和 `content` 属性，用CSS插入清除浮动的元素

后来有人发现了更简单的方式，仅仅使用 `overflow` 属性就可以。先看效果：
<iframe width="100%" height="300" src="http://jsfiddle.net/zicai/z5a34/7/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>
`overflow` 属性规定当内容溢出容器框时应该怎样。在默认情况下，内容会溢出到容器之外，进入相邻的空间。当 `overflow` 的属性值为 `hidden` 、`auto` 或 `scroll` 时有一个有用的副作用，就是它会自动清理包含的任何浮动元素。

`overflow`  这一副作用在大多数浏览器都有效，但是对于IE6、7我们仍需利用其 *auto-enclosing*（[上一篇有介绍](http://www.html-js.com/article/1691)），即需要触发容器的*hasLayout*。（注意：`overflow` 在IE7中可以触发*hasLayout*，在IE6中不可以）

所以第三种清除浮动的方法如下：

    .container {
        /*省略其它样式*/
    	overflow: hidden; /*为什么不用auto?，见下面说明*/
    	width: 100%; /*用于触发IE6下 hasLayout 。也可以使用其它可以触发hasLayout的属性*/
    }

需要注意的是：

 1. 对于opera浏览器则必须指定容器的`width`或者`height`。
 2. 对于Mac上的IE浏览器则必须使用`overflow:hidden` ，因为`overflow:auto`
    也会像`overflow:scroll`一样，总是产生滚动条。

但是这种方法也并不完美，先看效果：
<iframe width="100%" height="300" src="http://jsfiddle.net/zicai/z5a34/8/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>
我在内容里放置了一个相对定位的正方形，并试图将其定位到容器之外，结果它被切掉了一部分，因为该方法用到了`overflow:hidden`。
（当然，如果你的设计稿没有这种布局效果的话，那么是可以使用第三种方法的。）

再后来，有人改进了第二种清除浮动的方法，得到如下方法：

    .clearfix:before,
    .clearfix:after {
        content: " ";
        display: table;
    }
    .clearfix:after {
        clear: both;
    }
    .clearfix {
        *zoom: 1;/*For IE 6/7 触发hasLayout*/
    }

这种方法被称作*micro clearfix*，目前bootstrap 就采用这种方式清除浮动。

该方法使用`content:" "`代替了方法二中的`content:"."`，这样就没必要用`height: 0` 和`visibility: hidden`来隐藏生成的内容了（不仅减少了CSS代码量，而且避免了[Opera一个奇特的bug](http://nicolasgallagher.com/micro-clearfix-hack/#comment-40387)）。同方法二相同，使用`:after`伪类插入内容用于清除浮动。与之不同的是，使用`:before`伪类并且将生成的内容`display: table`是为了防止 *top-margin* 折叠。这样做的好处以及*top-margin* 折叠问题我们留到下篇再说。

至此我们说完了清除浮动的四种方式，似乎我们彻底弄明白了如何清除浮动。其实不然，在下一篇我会介绍你所不知道的【清除浮动】。是不是在想：啊！还有不知道的呢？。。哈哈，也许这就是CSS的博大精深吧。

参考文章：

 - [http://www.quirksmode.org/css/clearing.html](http://www.quirksmode.org/css/clearing.html)
 - [http://nicolasgallagher.com/micro-clearfix-hack/](http://nicolasgallagher.com/micro-clearfix-hack/)
 - [http://www.yuiblog.com/blog/2010/09/27/clearfix-reloaded-overflowhidden-demystified/](http://www.yuiblog.com/blog/2010/09/27/clearfix-reloaded-overflowhidden-demystified/)

题外话：

最近在看书的时候学到一句话和大家分享

> " I hear and I forget , I see and I remember , I do and I understand"

还看啥呢？动手吧！O(∩_∩)O


  [1]: http://jsfiddle.net/zicai/z5a34/8/embedded/