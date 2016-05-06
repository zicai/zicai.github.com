---
layout: post
category : lessons
title: "动画技术对比"
tagline: "Supporting tagline"
tags : [html]
---

我最常被问到的问题就是：你推荐哪个动画工具？


## 原生动画（Native Animation）
大多数第三方库底层使用的都是原生动画技术。

### CSS

#### 优势

- 性能好，尤其是开启了硬件加速。参见：[http://www.html5rocks.com/en/tutorials/speed/high-performance-animations/](http://www.html5rocks.com/en/tutorials/speed/high-performance-animations/)
- 使用原生 JavaScript 可以监听 `onAnimationEnd` 以及其它动画钩子
- otion along a path [is coming down the pipeline](https://codepen.io/danwilson/post/css-motion-paths) which will be rad.
- 通过媒体查询调整动画，进行响应式开发。

#### 劣势
- bezier easings 有些局限。你不能产出更复杂的物理效果来模拟更真实的运动
- If you go beyond chaining three animations in a row, I suggest moving to JavaScript. 通过 delay 来构造动画序列，有些复杂，如果调整时间，则要重新计算。
- 对沿着线运动的支持还不太好
- CSS + SVG animation has some strange quirkiness in behavior

### Canvas

Canvas 非常适合像素移动。利用它，你可以创建复杂的会话和交互，自带高性能渲染。

#### 优势

- 因为不是在 DOM 中移动，所以可以制作很复杂同时性能好的效果
- 交互很好
- 性能好
- 只有你想不到的，没有你做不到的

#### 劣势

- It’s difficult to make accessible.
- 不像 SVG 动画是 resolution-independent，Canvas 不适配 Retina。
- 不太容易调试

### SMIL

SMIL 是原生 SVG 动画规范。它允许你直接在 SVG DOM 中移动、渐变甚至交互。

it's losing support 

### Web Animation API

allows you to create more complex sequential animations without loading any external scripts.

#### 优势

- Sequencing is easy and super legible
#### 劣势

- 目前的支持还不太好


### WebGL

擅长交互和 3D 效果。

#### 优势

- 效果超炫
- 三维交互
- 硬件加速
- VR 的可能性

#### 劣势
- 不太容易学
- 响应式比较难

## 外部库（External Libraries）
### GreenSock（GSAP）
### VelocityJS
### jQuery
### Snap.svg
### Mo.js
### Three.js
### Bodymovin'

## 特定于 React 的工作流
### React-Motion
### GSAP 在 React
### CSS 在 React

## 结论


原文地址：

* https://css-tricks.com/comparison-animation-technologies

