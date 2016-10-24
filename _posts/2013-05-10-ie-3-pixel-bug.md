---
layout: post
title: "IE 3 像素 Bug"
description: ""
category: 
tags: []
---

## Bug 描述： ##
在**IE 6**中，与浮动元素相邻的非浮动元素，会与浮动元素之间存在3像素的间距。
## Bug 重现： ##
**测试用例1**代码如下：

    <div>
        <div style="float: left;width: 100px;height: 100px;border: 2px solid blue;"></div>
        <div style="border: 2px solid red;">
            Lorem ipsum dolor sit amet, consectetur adipisicing elit. Ab aliquid aspernatur assumenda autem consectetur eaque exercitationem 
        </div>
    </div>

显示效果如下：

现代浏览器 | IE 6 
--------------|:------:
![enter image description here][1] | ![enter image description here][2]

仔细对比上面的两张图，你会发现在现代浏览器中红框的内容是紧紧贴着蓝色边框的，而在IE6中，红框的内容与蓝色边框之间是有间隙的，这个间隙就是**3px**。

到底在什么情况下会出现这3px呢？会不会和红框`div`有关系？我们进一步简化**测试用例1**的代码，把红框`div`去掉，让文字内容直接跟在浮动元素之后。

**测试用例2**代码如下：

    <div style="float: left;width: 100px; height: 100px;border: 2px solid blue;"></div>
        Lorem ipsum dolor sit amet, consectetur adipisicing elit. Ab aliquid aspernatur assumenda autem consectetur eaque exercitationem

显示效果如下：

现代浏览器 | IE 6 
--------------|:------:
![enter image description here][3] | ![enter image description here][4]

发现3px 依旧存在。那我们继续修改测试用例，看看给文字内容与浮动框之间添加间距，不让它们直接接触的情况下，3px是不是会消失。

**测试用例3**代码如下：

    <div style="float: left;width: 100px; height: 100px;border: 2px solid blue;"></div>
        <div style="border: 2px solid red;margin-left:  120px;">
            Lorem ipsum dolor sit amet, consectetur adipisicing elit. Ab aliquid aspernatur assumenda autem consectetur eaque exercitationem
            Lorem ipsum dolor sit amet, consectetur adipisicing elit. Ab aliquid aspernatur assumenda autem consectetur eaque exercitationem
            Lorem ipsum dolor sit amet, consectetur adipisicing elit. Ab aliquid aspernatur assumenda autem consectetur eaque exercitationem
        </div>

其实就是在**测试用例1**的基础上，给红框添加了`margin-left:  120px`，使红框不再紧靠着蓝框，而是与蓝框保持一些距离。显示效果如下：

现代浏览器 | IE 6 
--------------|:------:
![enter image description here][5] | ![enter image description here][6]

我们发现即使不直接接触浮动元素，IE 6中还是存在3px。但是，请仔细观察IE 6中的效果图，发现什么问题没？或者看下面的局部放大图：

![enter image description here][7]

我们发现，红框里超出浮动元素高度的那部分内容与红边框之间是没有3px间距的。看到这，能想出IE 6到底是给谁加上的3px么？ 

## Bug 分析： ##

![enter image description here][8]

很明显：**IE 6给与浮动元素相邻的[line box](http://www.html-js.com/article/1645#line-box)添加了3px间距，而且，只有在浮动元素高度范围之内line box 才会被添加3px 间距。**


## 解决方案 ：##
解决方案第一步是让红框[拥有layout](http://www.html-js.com/article/1725)（为了简单起见，这里用`zoom:1`来使其拥有layout）

**测试用例4**代码如下：


        <div style="float: left;width: 100px;height: 100px;border: 2px solid blue;"></div>
        <div style="border: 2px solid red;zoom:1;">
            Lorem ipsum dolor sit amet, consectetur adipisicing elit. Ab aliquid aspernatur assumenda autem consectetur eaque exercitationem 
        </div>


显示效果如下：

现代浏览器 | IE 6 
--------------|:------:
![enter image description here][9] | ![enter image description here][10]

差别还是挺大的，我们一个个的分析。

先看现代浏览器，要想让红框不再与浮动元素重叠有两种选择：1，让红框产生BFC（有关BFC的内容以后详细说）。2，给红框添加`margin-left:104px`（需要把边框宽度`4px`考虑进去）。这里我们采用第二种方法。注意：这里的`margin-left:104px`在IE6中同样会起作用：使红框的左边框距离蓝框的左边框`104px`。

再看IE 6效果图，这次红框的内容与红色边框之间的3px消失了，可是红框与蓝框之间多了3px间距。我们要增加一条针对IE6的规则：给浮动元素添加`margin-right:-3px`（写到条件注释里）。而`margin-right:-3px`起作用的前提是：红框的`margin-left`值小于`104px`。

**测试用例5**代码如下：
<iframe width="100%" height="300" src="http://jsfiddle.net/zicai/VytQ3/1/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

显示效果如下：

现代浏览器 | IE 6 
--------------|:------:
![enter image description here][11] | ![enter image description here][12]

终于一致了。**总结下解决方案**：

 1. 触发红框hasLayout
 2. 给红框添加`margin-left`，并在IE6中取消`margin-left`带来的影响
 3. 给浮动元素添加`margin-right:-3px`(仅针对IE6)

参考文章：

 - [http://www.positioniseverything.net/explorer/threepxtest.html](http://www.positioniseverything.net/explorer/threepxtest.html)






  [1]: http://htmljs.b0.upaiyun.com/uploads/1389077715886-1.PNG
  [2]: http://htmljs.b0.upaiyun.com/uploads/1389077733118-2.PNG
  [3]: http://htmljs.b0.upaiyun.com/uploads/1389078325175-3.PNG
  [4]: http://htmljs.b0.upaiyun.com/uploads/1389078335116-4.PNG
  [5]: http://htmljs.b0.upaiyun.com/uploads/1389081318760-5.PNG
  [6]: http://htmljs.b0.upaiyun.com/uploads/1389081328423-6.PNG
  [7]: http://htmljs.b0.upaiyun.com/uploads/1389082746428-7.PNG
  [8]: http://htmljs.b0.upaiyun.com/uploads/1389084810711-8.png
  [9]: http://htmljs.b0.upaiyun.com/uploads/1389090679563-10.PNG
  [10]: http://htmljs.b0.upaiyun.com/uploads/1389090690795-9.PNG
  [11]: http://htmljs.b0.upaiyun.com/uploads/1389153750373-11.PNG
  [12]: http://htmljs.b0.upaiyun.com/uploads/1389153760996-12.PNG