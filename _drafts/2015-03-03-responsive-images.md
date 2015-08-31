---
layout: post
category : lessons
title: "响应式图片"
tagline: ""
tags : [响应式]
---

##介绍

响应式图片是指图片可以适应不同环境状态，适应的形式包括但不限于改变大小，裁切，甚至是改变图片源。

许多媒体特征（media features）经常都是变化的（例如：改变浏览器窗口大小，设备从竖向旋转为横向等等），用户代理（user agent）会不断地响应这些改变媒体特征的事件。文档布局会适应媒体特征的改变，图片的表达能力会被极大地降低（图片会压缩失真）。所以开发人员就需要依赖一些客户端或者服务端的解决方案来适应媒体特征的变化。

###建议的解决方案
- srcset 属性
    允许定义多个图片资源和暗示（hints）用来协助用户代理（通常是浏览器）决定显示哪个资源。

- picture 元素
    基于srcset，
- HTTP Client Hints
    定义一组HTTP header，允许浏览器表明

##当前技术的局限

##使用场景

###基于分辨率
###基于viewport
###基于Device-pixel-ratio-based
###Art direction
###Design breakpoints
###Matching media features and media types
###Relative units
###Image formats
###User control over sources