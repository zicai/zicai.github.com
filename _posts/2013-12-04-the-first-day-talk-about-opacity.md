---
layout: post
title: "第一天，谈谈【透明】"
description: ""
category: 
tags: []
---
{% include JB/setup %}
提到透明，大家首先会想到`opacity`。先来看测试用例一：

    .case{
		/*省略无关样式*/
		background-color: red;
		opacity: 0.5;
	}

在支持`opacity`的浏览器里效果如下：

![opacity in chrome][1]

`opacity`的兼容性如下图：

![opacity compatible][2]

在不支持`opacity`的IE6、7、8中效果如下：

![opacity in ie 8][3]

为了兼容IE6、7、8，需要使用`filter:alpha(opacity=..)`，来看测试用例二：

    .case{
		/*省略无关样式*/
		background-color: red;
		filter:alpha(opacity=50);/* for IE8 and earlier */
		opacity: 0.5;/* for IE9 and other browsers */
	}
	
在IE6中效果如下，（IE7、8也相同）

![opacity in ie6 fixed][4]

但是`opacity`会引起一些副作用，来看测试用例三：

    .case{
		/*省略无关样式*/
		background-color: red;
		filter:alpha(opacity=50);
		opacity: 0.5;
	}
	.inner{
		/*省略无关样式*/
		background-color: blue;
		color: #000;
	}
	
效果如下：（仔细看，最内层元素里是有文字的）

![opacity in chrome inner][5]

可以看到，透明元素包含的子元素也透明了，尽管已经将子元素的`color`设置为`#000`，但文字还是透明到看不清楚。

这很可能不是你想要的效果，那么为了避免子元素受影响，我们需要用到CSS3提供的`RGBA(R,G,B,A)`，
通过最后的参数A（Alpha通道），可以设置元素的透明度。测试用例四：

    .case{
		/*省略无关样式*/
		border:1px solid #000;
		background:rgba(255,0,0,0.5);
	}
	
显示效果如下：

![rgba in chrome][6]

但是在IE6、7、8中显示如下：

![rgba in ie6][7]

`RGBA(R,G,B,A)`的兼容性如下：

![rgba compatible][8]

为了兼容IE6、7、8，我们需要用到`DXImageTransform.Microsoft.gradient`滤镜，来看测试用例五：

    .case{
		/*省略无关样式*/
		border:1px solid #000;
		filter:progid:DXImageTransform.Microsoft.gradient(startColorstr='#7FFF0000', endColorstr='#7FFF0000');
		background:rgba(255,0,0,0.5);
	}
	
此时在IE6中效果如下：

![rgba ie 6 fixed][9]

简单解释下：要给`DXImageTransform.Microsoft.gradient`滤镜设置起点色（`startColorstr`）和终点色（`endColorstr`），在此我们只想让它透明而无需渐变，只需让起点色和终点色相同。需要注意的是，它们的取值是一个八位的十六进制值，前两位表示alpha通道值，00表示完全透明，FF表示完全不透明；后六位则是这个颜色的RGB值。

我们再次给透明元素添加一个子元素，测试用例六：

    .case{
		/*省略无关样式*/
		border:1px solid #000;
		filter:progid:DXImageTransform.Microsoft.gradient(startColorstr='#7FFF0000', endColorstr='#7FFF0000');
		background:rgba(255,0,0,0.5);
	}
    .inner {
		/*省略无关样式*/
		color: #000;
		background-color: blue;
	}
	
效果如下：（IE7、8相同）

![rgba-ie6-fixed-inner][10]

但是IE9却稍有不同，如下：

![enter image description here][11]

我们可以发现，在IE9中透明元素的颜色与其他浏览器稍有偏差，原因我们以后再探究。

为了保险起见，我们还可以增加一个fallback（后备）颜色，也就是不透明时的颜色，如果某个浏览器不支持透明，就会显示这个颜色。代码如下：

    .case{
		/*省略无关样式*/
		border:1px solid #000;
		background-color: #ff0000;
		filter:progid:DXImageTransform.Microsoft.gradient(startColorstr='#7FFF0000', endColorstr='#7FFF0000');
		background-color:rgba(255,0,0,0.5);
	}
	
但是惊奇的发现，增加后备颜色后，在IE6、7下显示正常，在IE8下却不正常了。

![rgba-ie6-fixed-inner-fallback][12]
![rgba-ie7-fixed-inner-fallback][13]
![rgba-ie8-fixed-inner-fallback][14]

这里需要一个小技巧，需要在样式里增加一条`background:transparent`，再看IE8效果：

![enter image description here][15]

至于为什么`background:transparent`会有这样的效果，我们后面再探究。

至此我们可以总结下，使元素透明的样式可以这样写：（颜色替换成你需要的）

> 推荐一个小工具：[css 背景颜色属性值转换][16]

    .case{
		/*省略无关样式*/
		background-color: #ff0000;
		background: transparent;
		filter:progid:DXImageTransform.Microsoft.gradient(startColorstr='#7FFF0000', endColorstr='#7FFF0000');
		background-color:rgba(255,0,0,0.5);
	}

在写上文的过程中，发现需要展开写的地方有很多，比方说：`RGBA(R,G,B,A)`，不仅仅可以用来背景透明，还有其它好多用途。还有`filter`也值得展开说说，突然觉得`transparent`好深奥，也得深入下。O(∩_∩)O哈哈~，没关系，慢慢来，后面会把它们一一展开。

那接下来准备谈谈`RGBA(R,G,B,A)`

如果你发现文章有错误，却不告诉我，那你也太不够意思了吧。哪怕是有疑问，也得告诉俺一声儿，咱可以一块研究研究嘛，对不？你说呢？O(∩_∩)O哈哈~

邮箱：395217502@qq.com

[我的豆瓣][17]


  [1]: http://htmljs.b0.upaiyun.com/uploads/1386064479852-opacity-chrome.PNG
  [2]: http://htmljs.b0.upaiyun.com/uploads/1386064578091-opacity-compatible.PNG
  [3]: http://htmljs.b0.upaiyun.com/uploads/1386064563208-opacity-ie7.PNG
  [4]: http://htmljs.b0.upaiyun.com/uploads/1386064597909-opacity-ie6-fixed.PNG
  [5]: http://htmljs.b0.upaiyun.com/uploads/1386065298517-opacity-chrome-inner.PNG
  [6]: http://htmljs.b0.upaiyun.com/uploads/1386067853879-rgba-chrome.PNG
  [7]: http://htmljs.b0.upaiyun.com/uploads/1386068008168-rgba-ie6.PNG
  [8]: http://htmljs.b0.upaiyun.com/uploads/1386069036901-rgba-compatible.PNG
  [9]: http://htmljs.b0.upaiyun.com/uploads/1386069432168-rgba-ie6-fixed.PNG
  [10]: http://htmljs.b0.upaiyun.com/uploads/1386071571127-rgba-ie6-fixed-inner.PNG
  [11]: http://htmljs.b0.upaiyun.com/uploads/1386071754786-rgba-ie9-fixed-inner.PNG
  [12]: http://htmljs.b0.upaiyun.com/uploads/1386075487197-rgba-ie6-fixed-inner-fallback.PNG
  [13]: http://htmljs.b0.upaiyun.com/uploads/1386075793104-rgba-ie7-fixed-inner-fallback.PNG
  [14]: http://htmljs.b0.upaiyun.com/uploads/1386075523993-rgba-ie8-fixed-inner-fallback.PNG
  [15]: http://htmljs.b0.upaiyun.com/uploads/1386075742193-rgba-ie8-fixed-inner-fallback-fixed.PNG
  [16]: http://www.linxz.de/demo/hex_color.html
  [17]: http://www.douban.com/people/56880223/