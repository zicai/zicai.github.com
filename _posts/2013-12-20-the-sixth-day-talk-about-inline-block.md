---
layout: post
title: "第六天，谈谈【inline-block】"
description: ""
category: 
tags: []
---

对一个html元素应用`display:inline-block`会产生怎样的效果？

[官方文档](http://www.w3.org/TR/CSS21/visuren.html#display-prop)是这样说的：

> **inline-block** 
> 
> This value causes an element to generate an inline-level
> block container. The inside of an inline-block is formatted as a block
> box, and the element itself is formatted as an atomic inline-level
> box.

也就是说：`inline-block`会使元素成为一个行级块容器。即在容器内部产生BFC，而容器本身则被格式化为一个行级元素（内联元素）。

更通俗的说就是：`inline-block`元素可以像块元素那样设置宽高等属性，同时还会像内联元素一样水平排列、可以设置`vertical-align`、收缩包围自身内容。

先来看看在现代浏览器中分别对块元素和内联元素应用`display:inline-block`后的效果。应用之前如下：
<iframe width="100%" height="300" src="http://jsfiddle.net/zicai/V2LMF/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>
接着我们给块元素和内联元素应用`display:inline-block`，效果如下：
<iframe width="100%" height="300" src="http://jsfiddle.net/zicai/V2LMF/1/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>
请比较上面的两组代码，和显示效果。

那么IE浏览器是否支持`inline-block`呢？答案是肯定的，从[MSDN的文档上](http://msdn.microsoft.com/zh-cn/library/ie/ms530751(v=vs.85).aspx)我们可以看到：

> The inline-block value is supported starting with Internet Explorer
> 5.5.

也就是说IE 从5.5版本就开始支持`inline-block`了。但是在IE 6、7中对元素应用`display:inline-block`产生的效果和现代浏览器并不完全一致。

接下来我们分别讨论在IE 6、7中对**内联元素**和 **块元素** 应用`display:inline-block`，看看会产生怎样的效果。

 - 给**内联元素**应用`display:inline-block`

 测试用例一如下：

        Lorem ipsum dolor sit amet, consectetur
        <span style="background: yellow;border: 2px solid blue;">Lorem ipsum <br> dolor sit amet. </span>
        adipisicing elit. Consequuntur a.

 给`span`应用`display:inline-block`前后的效果对比如下：

   应用前 | 应用后
   --------- | ---------
   ![inline-block][1] | ![enter image description here][2]

 在[上一篇有关hasLayout的文章](http://www.html-js.com/article/A-day-to-learn-CSS-and-on-the-fourth-day-talk-about-hasLayout)里，我们说过`display:inline-block`可以使元素拥有Layout。而拥有layout的元素表现出来的特征就是一个独立的矩形容器，并且其内容不受周围元素的影响。

 紧接着，还是使用测试用例一的代码，不过这次我们使用`zoom:1`，来使`span`拥有layout。对比效果如下：

应用 `zoom:1` 前 | 应用 `zoom:1` 后
--------- | ---------
![inline-block][3] | ![enter image description here][4]

 我们发现，使用`zoom:1`的效果和使用`display:inline-block`的效果是一样的。**事实上，在IE6、7中，只要使内联元素拥有layout（包括但不限于通过`display:inline-block`来获得Layout），内联元素就会表现的如同`inline-block`元素一样。**

 - 给**块元素**应用`display:inline-block`
 
 测试用例二如下：

        Lorem ipsum dolor
        <div class="block">Lorem ipsum <br> dolor sit amet.</div>
        sit amet  elit. Consequuntur a.
 给`div`应用`display:inline-block`前后的效果对比如下：

应用前 | 应用后
--------- | ---------
![enter image description here][5] |  ![enter image description here][6]

 看上去没有任何变化。其实，在IE6、7中，`display:inline-block`同样会使块元素获得layout，但是并不能使块元素表现的如同`inline-block`元素一样。

 通过上面的分析我们知道：在IE6、7中，让一个`inline`元素拥有Layout就可以使其表现的如同`inline-block`元素一样，那么相应的，我们让一个拥有Layout的元素`display:inline`会产生怎样的效果呢？

 看测试用例三：
 
 HTML：

        Lorem ipsum dolor
        <div class="block">Lorem ipsum <br> dolor sit amet.</div>
        sit amet elit. Consequuntur a.

 CSS：

        .block {
            background: red;
            border: 2px solid blue;
            display: inline-block;
        }
        .block {
            display: inline;
        }
 效果如下：

 ![enter image description here][7]

 我们通过`display: inline-block`使`div`拥有layout，随后又将其`display: inline`，这时块元素表现的就和`inline-block`元素一致了。需要注意的是：通过`display: inline-block`获得的Layout并不会因为`display: inline`而失去，详见[谈谈hasLayout](http://www.html-js.com/article/A-day-to-learn-CSS-and-on-the-fourth-day-talk-about-hasLayout)。而不会失去layout的前提是：将`display: inline-block`和
`display: inline`写到两条规则里，也就是说如果你写成下面这样是不行的：

        .block {
            background: red;
            border: 2px solid blue;
            display: inline-block;
            display: inline; /*写到了一条规则里*/
        }
 因为这样写的话，在`display: inline-block`使`div`拥有Layout之前就被`display: inline`覆盖了，即`div`根本没有得到layout。


 同时为了保证浏览器兼容性，需要将`display: inline`写到针对IE6、7的条件注释或hack中去，否则在现代浏览器中就会产生如下效果：

 ![enter image description here][8]

在阅读别人代码时，你可能会经常见到下面这样的写法：

    .XX{
        display: inline-block;
        *display: inline; 
        *zoom:1;
    }

上面的代码同样是为了让`XX`元素成为`inline-block`元素。现代浏览器只会识别`display: inline-block`，而IE6、7则会识别全部三条样式。通过上面的叙述，我想你应该明白为什么需要`*zoom:1`，这里不再赘述。

**总结一下：**

**在IE6、7中，`display: inline-block`会使元素拥有Layout。拥有Layout的元素如果同时`display: inline`，那么该元素就会表现得如同现代浏览器中的`inline-block`元素一样。**

参考文章：
 
 - [http://www.brunildo.org/test/InlineBlockLayout.html](http://www.brunildo.org/test/InlineBlockLayout.html)(需翻墙)



  [1]: http://htmljs.b0.upaiyun.com/uploads/1389517042358-1.PNG
  [2]: http://htmljs.b0.upaiyun.com/uploads/1389517061416-2.PNG
  [3]: http://htmljs.b0.upaiyun.com/uploads/1389517042358-1.PNG
  [4]: http://htmljs.b0.upaiyun.com/uploads/1389517061416-2.PNG
  [5]: http://htmljs.b0.upaiyun.com/uploads/1389521040870-3.PNG
  [6]: http://htmljs.b0.upaiyun.com/uploads/1389521087093-4.PNG
  [7]: http://htmljs.b0.upaiyun.com/uploads/1389521889987-5.PNG
  [8]: http://htmljs.b0.upaiyun.com/uploads/1389522296150-6.PNG