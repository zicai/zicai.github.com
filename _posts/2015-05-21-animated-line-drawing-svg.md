---
layout: post
category : lessons
title: "SVG动画--线绘"
tagline: "Supporting tagline"
tags : [svg]
---

## SVG里的`path`

SVG里`path`定义的格式和正则表达式一样难看。

```
<path fill="none" stroke="deeppink" stroke-width="14" stroke-miterlimit="0"
  d="M11.6 269s-19.7-42.4 6.06-68.2 48.5-6.06 59.1 12.1l-3.03 28.8 209-227s45.5-21.2 60.6 1.52c15.2 22.7-3.03 47-3.03 47l-225 229s33.1-12 48.5 7.58c50 63.6-50 97-62.1 37.9"
/>

```

`d`属性的每一部分通知渲染器：移动到某个点，开始画线，画贝塞尔曲线到另一点等等。

我们不能期望通过修改这些数据来实现动画的效果。一个取巧的方法是通过 make an SVG path dashed and control the offset of the dash，来实现动画。

```
<path stroke="#000" stroke-width="4.3" fill="none" d="…"
  stroke-dasharray="988.00 988.00"
  stroke-dashoffset="988.00"
/>
```

`stroke-dasharray` 属性用来控制绘制线条的短线和间隔的模式。可直接用在CSS中：

```
.path{
	stroke-dasharray:none|<dasharray>|inherit
}
```

`<dasharray>`是一个长度或者百分比的列表，用逗号或空格分隔。用于指定交替的短线和间隔的长度。如果提供了奇数个值，那么会重复到偶数。例如： 5,3,2 等同于 5,3,2,5,3,2

`stroke-dashoffset` 属性用于指定起始短线的偏离距离。值可以是负值。


```
.path{
	stroke-dashoffset:<percentage>|<length>|inherit
}
```


假设`.path`线长`500px`，想象一下，下面的CSS的效果：

```
.path{
	stroke-dasharray:500px 500px;
	stroke-dashoffset:500px;
}
```

然后我们把`stroke-dashoffset`的值逐渐减小到`0`（其实就是线条先偏离`500px`,然后慢慢回退到起点，就实现了动画效果）

那么该如何获取`path`的长度呢？利用js:

```
var path = document.querySelector('.somepath');
path.getTotalLength(); // 988.0062255859375
```

## 动起来

SVG动画最简单的方式就是使用CSS 的 animations或transitions。缺点就是不支持IE。如果打算支持IE，需要使用[`requestAnimationFrame`](https://developer.mozilla.org/en-US/docs/Web/API/window.requestAnimationFrame)

避免使用SMIL，IE不支持，同时它在Chrome和Safari中表现也不好。

### 使用transition：

```
var path = document.querySelector('#Path-1');
var length = path.getTotalLength();
// Clear any previous transition
path.style.transition = path.style.WebkitTransition =
  'none';
// Set up the starting positions
path.style.strokeDasharray = length + ' ' + length;
path.style.strokeDashoffset = length;
// Trigger a layout so styles are calculated & the browser
// picks up the starting position before animating
path.getBoundingClientRect();
// Define our transition
path.style.transition = path.style.WebkitTransition =
  'stroke-dashoffset 2s ease-in-out';
// Go!
path.style.strokeDashoffset = '0';
```

解释下上面的代码：`getBoundingClientRect`是个hack，用于触发layout。因为如果要在同一个javascript执行环境中修改同一个样式两次，只有最后一次会生效，除非强制在两次修改之间触发layout。详情参见：[http://www.smashingmagazine.com/2013/03/animating-web-gonna-need-bigger-api/](http://www.smashingmagazine.com/2013/03/animating-web-gonna-need-bigger-api/)


### 使用animation

```
<script>
    var path = document.querySelector('#Path-1');
    var length = path.getTotalLength();
    // Set up the starting positions
    path.style.strokeDasharray = length + ' ' + length;
    path.style.strokeDashoffset = length;
</script>

<style>
    #Path-1 {
        animation: dash 5s linear alternate forwards;
    }

    @keyframes dash {
        to {
            stroke-dashoffset: 0;
        }
    }
</style>
```


参考地址：

- [https://jakearchibald.com/2013/animated-line-drawing-svg/](https://jakearchibald.com/2013/animated-line-drawing-svg/)
- [https://css-tricks.com/svg-line-animation-works/](https://css-tricks.com/svg-line-animation-works/)