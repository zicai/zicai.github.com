---
layout: post
title: "第八天，谈谈【旋转和翻转】"
description: ""
category: 
tags: []
---

在切图过程中，经常会碰到一些小图标，比如各种方向的箭头等。这时我们就可以利用旋转和翻转，来减少切图工作量，同时也能减小图片体积。

我们有下面的这个图标，怎么得到其他方向的箭头呢？

![enter image description here][1]

旋转
--

CSS3提供了`transform`属性，和2D旋转函数`rotate()`

    transform: rotate(angle);       /* an <angle>, e.g., rotate(30deg) */

其中`angle`是旋转的角度，如果值为正数，则表示顺时针旋转，若值为负数，则表示逆时针旋转。例如：让元素顺时针旋转30度应该写成`transform: rotate(30deg)`。当然，由于`transform`属性还处于试验阶段，所以还需加上浏览器的私有前缀。

如何用上面的图标，得到向右的箭头呢？测试用例一如下：
<iframe width="100%" height="300" src="http://jsfiddle.net/zicai/3DBE9/1/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>
改变旋转的角度，你可以得到任意方向的箭头。

和`transform`密切相关的属性是`transform-origin`，它用来设置图形变换的原点。对`rotate`函数而言就是旋转的中心。在2D变换中，`transform-origin`的默认值是`50% 50%`，即以元素的中心作为旋转的圆心。

为了兼容IE6、7、8，需要使用微软特有的`filter`技术。分为两种情况：

 1. 旋转90度的倍数，使用`BasicImage`滤镜（简单）

        filter:progid:DXImageTransform.Microsoft.BasicImage(rotation=1);/*旋转90度*/

 `rotation`可取值0、1、2、3。默认值是**0**。当`rotation=1`时，旋转**90**度；`rotation=2`时，旋转**180**度；`rotation=3`时，旋转**270**度。

 2. 旋转任意角度，使用`Matrix`滤镜（稍复杂）

        filter: progid:DXImageTransform.Microsoft.Matrix(M11=v1,M12=v2,M21=v3,M22=v4,SizingMethod='auto expand');

 不要被这一长串参数吓到，其实很简单。我们只需给 `M11、M12、M21、M22` 这四个参数赋值即可。参数值怎么算呢？假设要使图形旋转X度，那么参数值分别为 `v1=cos(X)、v2=-sin(X)、v3=sin(X)、v4=cos(X)`。最后一个参数`SizingMethod`用来确定元素大小如何变化，可取值为`'clip to original'`和`'auto expand'`。`'clip to original'`表示元素大小不变，对旋转后的图形进行裁切。`'auto expand'`表示元素自动扩展大小以适应旋转后的图形。

翻转
--
同样是使用`transform`属性，但是CSS3并没有提供翻转函数。需要利用2D缩放函数`scale()`

    transform: scale(sx[, sy]);  /* one or two unitless <number>s, e.g., scale(2.1,4) */

`sx`代表X轴方向上缩放的倍数，`sy`代表Y轴方向上缩放的倍数，`sy`可有可无，如果没有的话，默认和`sx`相等。例如：想让元素长宽都缩小到原来的一半，可以写成`transform:scale(0.5)`。同样的，在实际应用时要加上浏览器私有前缀。

还可以只针对X轴方向进行缩放

    transform:  scaleX(sx)    /* a unitless <number>, e.g., scaleX(2.7) */

或只针对Y轴方向进行缩放

    transform:  scaleY(sy)    /* a unitless <number>, e.g., scaleY(0.3) */


当`sx`或`sy`为负值时，图像就会发生翻转。测试用例二如下：
<iframe width="100%" height="300" src="http://jsfiddle.net/zicai/3DBE9/3/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>
`sx`值为`-1`，图像发生水平翻转。得到的结果与测试用例一旋转180度后一致。但需要注意的是：翻转的结果并不一定和旋转180度的结果总相同。翻转以轴为镜像，旋转180度以点为镜像。体会一下。

同样，为了兼容IE6、7、8，还是使用`filter`

    filter:FlipH; /*Flip Horizontal  水平翻转*/
    filter:FlipV; /*Flip vertical 垂直翻转*/

总结起来就是:

        /*水平翻转*/
        .flipx {
            -webkit-transform: scaleX(-1);
            -moz-transform: scaleX(-1);
            -ms-transform: scaleX(-1);
            -o-transform: scaleX(-1);
            transform: scaleX(-1);
            filter: FlipH();
        }
        /*垂直翻转*/
        .flipy{
            -webkit-transform: scaleY(-1);
            -moz-transform: scaleY(-1);
            -ms-transform: scaleY(-1);
            -o-transform: scaleY(-1);
            transform: scaleY(-1);
            filter: FlipV();
        }

了解了旋转和翻转，以后切图的时候试着用一下。


  [1]: http://htmljs.b0.upaiyun.com/uploads/1392273296947-circle-backward-128.png
  [2]: http://jsfiddle.net/zicai/3DBE9/embedded/