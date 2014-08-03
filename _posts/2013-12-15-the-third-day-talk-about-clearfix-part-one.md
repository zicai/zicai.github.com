---
layout: post
title: "第三天，谈谈【清除浮动 (上)】"
description: ""
category: 
tags: []
---

初学CSS的时候，都会遇到下面这种情况:
<iframe width="100%" height="300" src="http://jsfiddle.net/zicai/z5a34/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>
我们期望的结果是`container`包裹住它的内容。但是结果`container`表现的如同浮动内容不存在一样。

原因得从CSS定位说起：

> CSS中有3种基本的定位机制：普通流、浮动和绝对定位。除非专门指定，否则所有的框都在普通流中定位。                -----------《精通CSS （第二版）》

我们没有指定`container`的定位方式，所以它在普通流中定位。而浮动元素则会**脱离**普通文档流，不再影响不浮动的元素，所以普通文档流中的块框（例如这里的`container`）表现的就像浮动框不存在一样。要想使`container`包裹住浮动元素，就需要对浮动进行清理，这就用到了`clear`属性。

###`clear`属性有什么作用呢？

> `clear` 属性规定元素的哪一侧不允许有其他浮动元素。

###`clear` 是如何起作用的呢？

> 在 CSS1 和 CSS2 中，是通过自动为清除元素（即设置了 clear 属性的元素）增加**上外边距**实现的。在 CSS2.1
> 中，会在元素**上外边距**之上增加**清除空间**，而外边距本身并不改变。两种清除方式，最终结果都一样，即如果声明为左边或右边清除，会使元素的上外边框边界刚好在该边上浮动元素的下外边距边界之下。

于是有了第一种最简单的清除浮动的方式：添加清除元素（带有`clear`属性的元素）
<iframe width="100%" height="300" src="http://jsfiddle.net/zicai/z5a34/4/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>
`clearElement`因为样式`clear:both`，所以它会出现在浮动元素之下。而作为`clearElement`父元素的`container`，为了包裹住`clearElement`，就需要拉伸自己高度。与此同时就造成一种包裹住浮动元素的**假象**（因为浮动元素已经脱离普通文档流，所以`container`是不可能真正包裹住浮动元素的，只是在高度上超过了浮动元素）。

上面这种清除浮动的方式是通过手动添加额外的html标签来实现的，那我们能否让程序自动添加额外内容来清除浮动呢？答案是可以的。可以用javascript动态添加内容（这里不展开介绍），还可以用CSS添加内容。

用CSS添加内容就用到了`content`属性：

> `content`属性与 `:before` 及 `:after` 伪元素配合使用，来插入生成内容

先来试下`content`属性的效果：
<iframe width="100%" height="300" src="http://jsfiddle.net/zicai/8tGZr/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>
这样我们就可以利用`content`属性插入的内容来清除浮动，第二种清除浮动的方法如下：

    .clearfix:after {
        content: ".";
        display: block;
        clear: both;
        height: 0;
        visibility: hidden;
    }

这种方法被称作`easyclearing`。给容器添加`clearfix` 类即可达到清除浮动的目的。做个简单的说明：首先用`content`属性插入了一个*句点*，默认情况下`content`属性插入的内容是行内元素，需要将其`display: block`，这样才能使 `clear: both`生效。为了不让插入的内容影响到原来的布局，需要通过 `height: 0`将其高度设为0，但高度设为0之后，插入的*句点*还是可见的，所以还需要设置其可见性`visibility: hidden`。

<a id="haslayout-clear-float"></a>
可是对于不支持`:after`伪类的IE6、7，还需要多做一步工作。请先在IE6、7中实验下面代码：

    <style>
        .container {
            background-color: #eee;
            border: 1px solid blue;
            zoom: 1; /*触发hasLayout*/
        }
        .left {
            float: left;
            width: 50px;
            height: 50px;
            background-color: yellow;
        }
        .right {
            float: right;
            width: 100px;
            height: 100px;
            background-color: red;
        }
    </style>
    <div class="container">
        <div class="left">左浮动</div>
        <div class="right">右浮动</div>
    </div>

会发现，我们并没有添加任何用来清除浮动的元素，但容器却也包裹住了浮动元素。这种称作 *float-enclosing* 或 *auto-enclosing* 的效果出现的前提是：触发容器的 *hasLayout* 。上面代码使用 `zoom: 1`来触发 *hasLayout* ，也可以使用其它可以触发 *hasLayout* 的属性。IE6、7这种不符合CSS规范的效果可以用来完善上面的方案。

    .clearfix:after {
          content: ".";
          display: block;
          clear: both;
          height: 0;
          visibility: hidden;
        }
    .clearfix{
          zoom:1; /* For IE6、7 触发 *hasLayout**/
    }

这样我们清除浮动的方法二就能兼容IE6、7了。

可是，后来又有人发现了更简单的清除浮动的方式。哈哈，留到下篇接着说（文章太长，不好╮(╯﹏╰）╭）。



参考文章：
[http://www.positioniseverything.net/easyclearing.html](http://www.positioniseverything.net/easyclearing.html)

  [1]: http://jsfiddle.net/zicai/8tGZr/embedded/