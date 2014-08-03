---
layout: post
title: "第二天，谈谈【line-height】"
description: ""
category: 
tags: []
---

先来看[MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/line-height)上的总结：
> - On block level elements, the line-height CSS property specifies the minimal height of line boxes within the element.
> - On non-replaced inline elements, line-height specifies the height that is used in the calculation of the line box height.
> - On replaced inline elements, like buttons or other input element, line-height has no effect.<sup>\[1]<sup>

也就是说：

 1. 在*block level elements*（块元素）上使用`line-height`，也就指定了块元素内部*line boxes*的最小高度
 2. 在*non-replaced inline elements*上使用`line-height`，`line-height`会被用来计算*line boxes*的高度。
 3. 在*replaced inline elements*上使用`line-height`没有效果。例如：`button`或者其它`input`标签。<sup>\[1]<sup>

第3条规则看起来最容易理解，我们先拿它来实验下，看测试用例一：

    .button-1{
        line-height: 20px;
    }
    .button-2{
        line-height: 30px;
    }
    .input-1{
        line-height: 20px;
    }
    .input-2{
        line-height: 30px;
    }
效果如下：

![enter image description here][1]

![enter image description here][2]

出人意料的是在*replaced inline elements*上使用`line-height`居然有效果，与第3条规则不符。注意到第3条规则是有注解的。还好，注解里给出了原因：
><sub>\[1]<sub> Neither engine implements the correct behavior with replaced inline elements like buttons. In some cases line-height is allowed to have an effect on them. This is incorrect behavior relative to the specification.

也就是说浏览器都没有正确的实现这一效果。按照CSS规范来看，浏览器在这一效果上都错了。

接着来看第2条规则：

 - 在*non-replaced inline elements*上使用`line-height`，`line-height`会被用来计算*line boxes*的高度。

至于到底如何计算，它并没有说明。在介绍计算规则之前，需要先明确几个概念。用代码说话：

    <p>
        The <em>emphasis</em> element is defined as "inline".
    </p>

显示效果如下：

![enter image description here][3]

上面的代码涉及到了四种盒子：

 1 . *block-level boxes*（块级盒），由代码中的`<p>`标签产生，它可以包含其它的*box*。
 
 ![enter image description here][4]

 2 . *inline-level boxes*（行级盒），在`<p>`标签中有一系列的*inline-level boxes*，如下：
 
![enter image description here][5]

 上面的*inline-level boxes*可以分为两类，一类是由内联元素产生的，例如`<em>`
 
![enter image description here][6]

而余下的两个没有被标签包裹的*inline-level boxes*就被称为*anonymous inline-level boxes*（匿名行级盒），如下：

![enter image description here][7]

 3 . *line box*，由*inline-level boxes*连接而成，每一行称为一个*line box*
 
![enter image description here][8]

 4 . *content area*，它是围绕着文字的并且看不见的一种*box*，它的高度取决于`font-size`。如下：
 
![enter image description here][9]

概念介绍完了，回到我们的问题上：

***non-replaced inline elements*如何根据`line-height`来计算*line boxes*的高度**

上面我们说过*line box*是由*inline-level boxes*连接而成，所以我们将问题拆成两步：

 1. 先探究`line-height`与*inline-level boxes*高度的关系
 2. 再看*inline-level boxes*的高度与*line box*高度的关系


先来看看`line-height`与*inline-level boxes*高度的关系，测试用例二：
<iframe width="100%" height="300" src="http://jsfiddle.net/zicai/DpU9F/5/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>
再结合下面这张图

![enter image description here][10]

你应该就能看明白了。

我们把`line-height: 30px`与`font-size: 20px`的差值`10px`称为行间距（英文是*leading*）。那么半行间距（英文是*half-leading*）就是` 10px / 2 = 5px `。半行间距作用在*content area*的顶部和底部。

![enter image description here][11]

很简单，对不对？

但是别忽略了，上面计算的前提是`line-height`大于`font-size`，那如果`line-height`小于`font-size`会怎么样呢？来看测试用例三：
<iframe width="100%" height="300" src="http://jsfiddle.net/zicai/Wbp2E/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>
我们看到当`line-height`小于`font-size`时，*inline-level boxes*会使用`line-height`作为自己的高度，此时 *content area* 会**溢出** *inline-level boxes* ，如下图所示：

![enter image description here][12]

那么如果将`line-height`设置为`0`会出现啥效果呢？我们以后再探究。

至此，我们说完了`line-height`与*inline-level boxes*高度的关系，接下来该看*inline-level boxes*的高度与*line box*高度的关系了。记住下面这条规则：

> *line box* 的高度由它内部 **最高的** *inline-level boxes* 或 *replaced element* 来决定。

来看测试用例四：
<iframe width="100%" height="300" src="http://jsfiddle.net/zicai/QGmPX/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>
至此，我们终于清楚了`line-height`与*line boxes*高度的关系，也就清楚了
[MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/line-height)上第2条规则的含义：

 - 在*non-replaced inline elements*上使用`line-height`，`line-height`会被用来计算*line boxes*的高度。

需要补充的一点是：*line boxes* 在它的容器里是紧挨着彼此堆叠到一起的，看下图会更好理解一些：

![enter image description here][13]

最后，我们来看第1条规则：

 - 在*block level elements*（块元素）上使用`line-height`，也就指定了块元素内部*line
   boxes*的最小高度

理解这一条需要一点想象力，看着下图，开动你的想象力：

![enter image description here][14]

想象在每一个*line box*的起始位置都有一个宽度为零的 *inline-level boxes* ，我们把这个想象出来的盒子叫做*strut*（这个名字来源于[*TeX*](http://zh.wikipedia.org/zh/TeX)）。*strut*会根据继承来的`font-size` 和 `line-height` 计算本身所占高度。这个高度也就是所在*line boxes*的最小高度。

参考文献：

 1. [http://www.w3.org/TR/2011/REC-CSS2-20110607/visudet.html#line-height](http://www.w3.org/TR/2011/REC-CSS2-20110607/visudet.html#line-height)
 2. [http://www.w3.org/TR/css3-box/](http://www.w3.org/TR/css3-box/)
 3. [CSS布局](http://www.cnblogs.com/winter-cn/archive/2013/05/11/3072929.html)
 4. [一个关于line-height非常好的PPT，本文部分图片来自这个PPT，需翻墙](http://www.slideshare.net/anuradhay_2004/css-line-height)

说了这么多有关`line-height`的问题，其实还有好多知识点没有涉及到，比如`line-height`的继承问题以及`0`行高问题等等。我们后面会一一探讨。




  [1]: http://htmljs.b0.upaiyun.com/uploads/1386228453943-1.PNG
  [2]: http://htmljs.b0.upaiyun.com/uploads/1386229490559-2.PNG
  [3]: http://htmljs.b0.upaiyun.com/uploads/1386242016916-3.PNG
  [4]: http://htmljs.b0.upaiyun.com/uploads/1386242489515-4.PNG
  [5]: http://htmljs.b0.upaiyun.com/uploads/1386242699209-5.PNG
  [6]: http://htmljs.b0.upaiyun.com/uploads/1386243055840-6.PNG
  [7]: http://htmljs.b0.upaiyun.com/uploads/1386243169014-7.PNG
  [8]: http://htmljs.b0.upaiyun.com/uploads/1386244313793-8.PNG
  [9]: http://htmljs.b0.upaiyun.com/uploads/1386245424193-9.PNG
  [10]: http://htmljs.b0.upaiyun.com/uploads/1386251530759-12.PNG
  [11]: http://htmljs.b0.upaiyun.com/uploads/1386252833411-13.PNG
  [12]: http://htmljs.b0.upaiyun.com/uploads/1386300675855-14.PNG
  [13]: http://htmljs.b0.upaiyun.com/uploads/1386304220478-15.PNG
  [14]: http://htmljs.b0.upaiyun.com/uploads/1386314817012-16.png