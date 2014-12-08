---
layout: post
title: "第五天，谈谈【hasLayout】"
description: ""
category: 
tags: []
---

##Layout是什么##
“Layout”是IE浏览器的私有概念，它决定了一个元素如何显示以及约束其包含的内容、如何与其他元素交互和建立联系、如何响应和传递应用程序事件/用户事件等。

##为什么需要Layout##
'Layout' 是 IE 浏览器渲染引擎的一个内部组成部分(注意，Layout 在 IE 8 及之后的 IE 版本中已经被抛弃，所以下面的讨论只针对IE6、7)。在 IE 浏览器中，一个元素要么是依赖祖先元素来管理自身尺寸和内容， 要么是自己负责管理自身尺寸和内容。为了协调这两种方式的矛盾，渲染引擎采用了 'hasLayout' 属性，属性值可以为 true 或 false。 当一个元素的 'hasLayout' 属性值为 true 时，我们说这个元素'hasLayout'，即拥有布局。

一个元素'hasLayout'就是指该元素自己负责管理自身尺寸和几乎所有子元素（除了那些同样拥有layout的子元素，因为这些拥有layout的子元素同样是自己负责管理自身的尺寸和后代）

那为什么不是所有元素都有layout？让所有元素都自己管理自己的尺寸和内容不好么？

微软给出的原因是“出于性能和简洁性考虑”。因为如果每个元素都缓存自己的布局信息会占用内存和计算时间。

##元素如何才能拥有layout##
有一些元素默认就拥有layout，例如：

 -  `<img>`
 -  `<table> <tr> <th> <td>`
 -  `<hr>`
 - `<input> <select> <textarea> <button>`
 - `<marquee>`
 - `<iframe>`
 - `<object> <applet> <embed>`
 - `<html> <body>`

其它元素在满足一定条件时也会拥有layout，例如：

 - `position:absolute`的元素
 - `float: left | right` 的元素
 - `display: inline-block`的元素
 - 在严格模式下，块级元素设置宽度/高度（值不能为`auto`）后会拥有layout
 - 在compat模式下，任何元素赋值宽度/高度（值不能为`auto`）后都会具备layout
 - 设置zoom属性（值不为`normal`）的元素具备layout
 - 跟父级元素的布局流不相同的元素(rtl to ltr)

当元素满足以上任何一个条件时在IE6、7中都会拥有layout。还有一些CSS属性只在IE7中能让元素拥有layout，例如：

 - `overflow: hidden | scroll | auto`的元素
 - `overflow-x|-y: hidden | scroll | auto` 的元素
 - `min-width | min-height: 任意值（包括0）`的元素
 - `max-width | max-height: 除 “none” 之外的任意值`的元素

##如何判断元素是否hasLayout##

通过脚本属性`hasLayout` 就可以判断元素是否拥有布局。假设有如下元素：

    <div id="test" style="zoom:1;">

在IE开发者工具控制台中通过如下代码即可查看元素是否拥有布局：

    alert(test.currentStyle.hasLayout)

注意：没有办法设置hasLayout=false， 除非把一开始那些触发hasLayout的CSS属性去除
。

##元素hasLayout之后会有怎样的变化##

 - 变化一：在拥有Layout的容器中，浮动元素参与容器高度的计算。

 在IE6、7中测试如下代码：


        <div id="container" style="border: 1px solid blue;">
            <div style="float: left; width: 100px; height: 100px; background: red;"></div>
        </div>

  效果如下：

![在拥有Layout的元素中，浮动元素参与高度的计算，触发之前][1]

我们给`container`添加样式`zoom:1`，使其拥有Layout，效果如下：

![在拥有Layout的元素中，浮动元素参与高度的计算，触发之后][2]

拥有Layout之前：由于浮动元素脱离文档流，不参与容器高度计算，所以容器高度为0。拥有Layout之后，按照规则，浮动元素就需要参与容器高度的计算。这也就是[为什么在IE6、7中可以利用hasLayout来清除浮动](http://www.html-js.com/article/1691#haslayout-clear-float)。

 - 变化二：与浮动元素相邻的，拥有Layout的元素不能与浮动元素相互覆盖。

 在IE6、7中测试如下代码：

        <div style="float: left;width: 30px;height: 30px;background: red;"></div>
        <div style="background: yellow;">Lorem ipsum dolor sit amet, consectetur adipisicing elit. Quae, reprehenderit!Lorem ipsum dolor sit amet, consectetur adipisicing elit. Iusto, rerum.</div>

 效果如下：

![与浮动元素相邻的，拥有Layout的元素不能与浮动元素相互覆盖。触发前][3]

黄色矩形与浮动的红色矩形发生了重叠。接着，我们给黄色矩形添加样式`zoom:1`，使其拥有Layout，效果如下：

![与浮动元素相邻的，拥有Layout的元素不能与浮动元素相互覆盖。触发后][4]

黄色矩形拥有Layout后就不能与浮动元素相互覆盖了。

 - 变化三：拥有layout的元素不会与它们的子元素发生外边距折叠。

 Collapsing Margins 即外边距折叠。参见[http://www.w3.org/TR/CSS21/box.html#collapsing-margins](http://www.w3.org/TR/CSS21/box.html#collapsing-margins)

 根据规范，一个盒子如果没有`padding-top`和`border-top`，那么它的上边距应该和其文档流中的第一个孩子元素的上边距重叠。

 在IE6、7中测试如下代码：

        <div id="container" style="border:2px solid blue;">
            <div id="father" style="background:yellow;">
                <div id="son" style="margin:30px 0; ">content</div>
            </div>
        </div>
 效果如下：

![拥有layout的元素不会与它们的子元素发生外边距折叠，触发前][5]

有人把上面这种情况叫做"劫持父元素"，因为看上去`son`元素的`margin-top`似乎冲破了`father`元素的界限，并迫使`father`元素下降了`30px`。要知道这种效果其实是符合CSS规范的。因为`father` 元素没有`padding-top`和`border-top`，所以按照规范它的上边距就应该和自己的第一个子元素上边距重叠。

 接着，我们给`father`元素添加样式`zoom:1`，使其拥有Layout，

 ![拥有layout的元素不会与它们的子元素发生外边距折叠，出发后][6]

拥有Layout的`father`元素就不再与子元素发生外边距折叠。

当然，元素拥有layout之后的变化不止这三点，还有很多变化与IE的bug有关，尤其是涉及到相对定位或者绝对定位。这些问题我们留到后面分析。

参考资料：

 - [On having layout](http://www.satzansatz.de/cssd/onhavinglayout.html)
 - ["HasLayout" Overview](http://msdn.microsoft.com/en-us/library/bb250481%28VS.85%29.aspx)
 - [http://www.w3help.org/zh-cn/causes/RM8002](http://www.w3help.org/zh-cn/causes/RM8002)



题外话：

依然是分享一段话：来自刘瑜的[《观念的水位》](http://book.douban.com/subject/20463108/)

> 我以前在国内读研上课时，可怜的老师时不时被学生这样质问：老师你说我们学这些有什么用呢？能不能教点对我们找工作有帮助的东西？

> 我很想知道当年牛顿讲授重力原理和月亮轨迹时，是不是也有一帮这么讨厌的人在问：老师你说我们学这些有什么用呢？而如果有人这样问，牛顿会不会反问：难到仅仅满足我们的好奇心还不够吗？

想想我自己上学时就曾这么被讨厌过，%>_<%  保持一颗好奇心，技术才会有进步，生活才不会无聊 ，嗯。


  [1]: http://htmljs.b0.upaiyun.com/uploads/1388905700116-1.PNG
  [2]: http://htmljs.b0.upaiyun.com/uploads/1388905888623-2.PNG
  [3]: http://htmljs.b0.upaiyun.com/uploads/1388907789817-3.PNG
  [4]: http://htmljs.b0.upaiyun.com/uploads/1388908494462-4.PNG
  [5]: http://htmljs.b0.upaiyun.com/uploads/1388910497208-5.PNG
  [6]: http://htmljs.b0.upaiyun.com/uploads/1388910521389-6.PNG