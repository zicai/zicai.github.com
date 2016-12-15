---
layout: post
category : svg
title: "理解 SVG 坐标系统"
tagline: "Supporting tagline"
tags : [svg]
---

一些名词：

- SVG canvas：、the space where the SVG content is rendered
- SVG viewport：SVG canvas is infinite for each dimension of the space, but rendering occurs relative to a finite rectangular region of the canvas. This finite rectangular region is called the SVG viewport


Viewport，viewbox，preserveAspectRatio

不受 CSS 盒模型的控制

控制 SVG 坐标系统最重要的三个属性：Viewport，viewbox，preserveAspectRatio

## SVG canvas

理论上画布是无限大。实际中，是在以一个有限的区域当中，也就是 SVG viewport

## SVG viewport

SVG viewport 类似于浏览器的 viewport.

通过最外层 `<svg>` 元素的 `width` 和 `height` 来指定 viewport 的尺寸。

```
<!-- the viewport will be 800px by 600px -->
<svg width="800" height="600">
    <!-- SVG content drawn onto the SVG canvas -->
</svg>
```

在 SVG 里，值可以有单位也可以没有单位。A unitless value is said to be specified in user space using user units.

一旦最外层 SVG 元素的宽高设定好了，浏览器就会确立一个初始 viewport 坐标系统和初始用户坐标系统

## 初始坐标系统

- 初始 viewport 坐标系统（也叫 viewport space）是在 viewport 上的坐标系统，原点在左上角，坐标(0,0)。正向 X 轴向右，正向 Y 轴向下。在初始坐标系统中一个 unit 等于 viewport 的一像素。
- 初始用户坐标系统（也叫当前坐标系统，或者 user space in use）是在 SVG canvas 上确立的坐标系统。初始状态和 viewport 坐标系统一致。可以通过 viewbox 属性修改用户坐标系统，使它和 viewport 坐标系统不一致。

使用 transform 属性可以创建新的 user space。

## viewBox
可以把 viewBox 看成真正的坐标系统。毕竟是用它来在画布上绘制 SVG 图形。

默认的用户坐标系统和 viewport 坐标系统一致。

使用 viewBox 属性可以声明自己的用户坐标系统。如果声明的坐标系统宽高比与 viewport 坐标系统一致，那么它会 stretch to fill the viewport area。如果不一致，可以使用 preserveAspectRatio 属性指定是否让整个坐标系统在 viewport 中出现，还可以指定如何定位。

首先，来看宽高比相等的情况，在这些例子中，preserveAspectRatio 无效。

在这之前，先来看下 viewBox 的语法

### viewbox 语法



```
viewBox = <min-x> <min-y> <width> <height>
```

`<min-x>`、`<min-y>` 用来定义 viewBox 左上角的位置，`<width> `、`<height>` 用来定义 viewBox 的尺寸。

例如：

```
<!-- The viewbox in this example is equal to the viewport, but it can be different -->
<svg width="800" height="600" viewbox="0 0 800 600">
    <!-- SVG content drawn onto the SVG canvas -->
</svg>
```

you can use the viewBox attribute to transform the SVG graphic by scaling or translating it. This is true. I’m going to go further and say that you can even crop the SVG graphic using viewBox.



### viewbox 宽高比等于 viewport 的宽高比

一个简单的例子。例子中，viewBox 宽高都是 viewport 的一半。也就是说宽高比相等。先不改变 viewBox 的原点，所以 `<min-x>`、`<min-y>` 都是 0。

```
<svg width="800" height="600" viewbox="0 0 400 300">
    <!-- SVG content drawn onto the SVG canvas -->
</svg>
```

`viewbox="0 0 400 300"` 做了哪些工作呢？

- 它在 canvas 上声明了一块区域，左上角(0,0)到右下角（400,300）
- 裁剪该区域内的图形
- 区域拉伸到匹配整个 viewport
- 用户坐标系统映射到 viewport 坐标系统--在本例中--一个用户单位等于两个 viewport 单位

你在 canvas 上画的任何图形都会相对于新的用户坐标系统。

想象下 Google map

接下来，改变 `<min-x>`、`<min-y>`，都设置为 100，


当宽高比不相同时

- 整个 viewBox fit viewport
- viewBox 的宽高比保留，没有拉伸
- viewBox 水平、垂直都居中在 viewport

这是默认的行为，谁控制的？如果想改变 viewBox 在 viewport 中的位置？这就引出了 preserveAspectRatio 属性。
## preserveAspectRatio 属性

The preserveAspectRatio attribute is used to force a uniform scaling for the purposes of preserving the aspect ratio of a graphic.

在保留宽高比的同时，强制统一缩放 viewBox。还允许你指定在 viewport 中如何放置 viewbox



### preserveAspectRatio 语法

```
preserveAspectRatio = defer? <align> <meetOrSlice>?
```

该属性在任何确立一个新的 viewport 元素上都可用。

defer 参数是可选的，只有当 image 时才使用

align 参数只是是否强制统一缩放，如果是，使用哪种对齐方式

如果 preserveAspectRatio 值为 none，则不保持宽高比，拉伸至匹配 viewport

preserveAspectRatio 其它值都会保持宽高比的前提下强制统一缩放。

最后一个参数，也是可选的，默认是 meet。用来指示是否要在 Viewport 内部显示整个 viewBox 。

你可以把 meetOrSlice 想象成 background-size 的值 contain 和 cover。meet 类似于 contain，而 slice 类似于 cover。

#### meet
在下面保持下面两个前提下，尽可能缩放图形：

- 保持宽高比
- 整个 viewBox 在 viewport 中都可见

#### slice
在保持宽高比的前提下，缩放图形保证 viewbox 覆盖整个 viewport 区域。不会做多余的缩放。

align 的值是九选一，或者是 none。

参考资料：

- [https://www.w3.org/TR/SVG11/coords.html](https://www.w3.org/TR/SVG11/coords.html)
- [https://sarasoueidan.com/blog/svg-coordinate-systems/](https://sarasoueidan.com/blog/svg-coordinate-systems/)
- http://www.w3cplus.com/html5/svg-coordinate-systems.html 中文








