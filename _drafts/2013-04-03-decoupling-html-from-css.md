---
layout: post
category : css
title: "decoupling-html-from-css"
tagline: "Supporting tagline"
tags : [css]
---
原文地址：[https://www.smashingmagazine.com/2012/04/decoupling-html-from-css/](https://www.smashingmagazine.com/2012/04/decoupling-html-from-css/)

多年以来，Web 标准社区已经讨论过关注点分离。将CSS从JavaScript 和html 中分离出来。CSS独立到文件、JavaScript独立到文件，HTML 也独立成文件。非常干净。

CSS zen garden 已经证明了我们可以通过仅替换CSS就更改网站主题。但是我们很少注意到分离之后的缺点：如果需要修改HTML，我们需要修改HTML 同时还要修改对应的CSS。

这样看来，实际上我们并没有真正分离。因为我们需要在两个地方进行修改。

## 探索解决方案

从个人的独立作战到雅虎的大团队，我开始思考痛点是什么以及如何避免？

先看其他人是如何做的。只举几个例子：Nicole Sullivan’s [Object-Oriented CSS](https://github.com/stubbornella/oocss/wiki),Jina Bolton’s [CSS Workflow](https://vimeo.com/15982903),Natalie Downe’s [Practical, Maintainable CSS](http://www.slideshare.net/nataliedowne/practical-maintainable-css)。

最后写下我自己的思考,一个长篇的样式规范，叫做“Scalable and Modular Architecture for CSS.“ 简称：SMACSS ，读作：smacks。

在探索的过程中，我发现，设计师写的CSS通常会与HTML结构紧紧耦合。那么该如何解耦，更灵活的开发以及减少重构。

换句话说：如何避免给所有规则都加上 !important 或是掉入选择器地狱。

## 重用样式

在最开始，我们给每个html元素应用单独的样式。这种方式很不实际，这样CSS诞生了。CSS允许我们在页面中不同元素间重用样式。



## 降低适用性

CSS 最常见的问题可能就是管理特殊性（specificity）。
多条CSS规则对同一个元素都起作用。

通过添加选择器，来增加特殊性（specificity）。

然而，随着项目复杂度的增加，这就变成了猫鼠游戏。相反的，我们应该限制CSS的影响范围。

在SMACSS中，我把这种影响称为  “depth of applicability.”。

适用性越强，影响就越大，HTML与CSS的耦合就越严重。

可维护的CSS的目标就是：限制适用性。换句话说，编写的CSS只对我们想应用的元素生效。

### 子选择器

### 


























