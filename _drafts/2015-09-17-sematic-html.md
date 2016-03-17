---
layout: post
category : lessons
title: "HTML 标签"
tagline: "Supporting tagline"
tags : [html]
---

## 基础元素

- `html`

## 文档元数据

- `<base>`:用来指定文档中所有相对路径的基础路径。一个文档只能包含一个`<base>`
- `<head>`
- `<link>`:指明当前文档和外部文档的关系。
- `<meta>`:用来表示其它 文档元数据标签 不能表示的元数据。
- `<style>`
- `<title>`

资源：
- [http://www.w3.org/TR/html5/links.html](http://www.w3.org/TR/html5/links.html)

## 内容块

- `<address>`:给最近的祖先元素<article> 或 <body> 提供联系信息
- `<article>`:表示自包含的内容，可单独发布，重用。可以是论坛帖子，文章，博客或是其它独立的内容。通常会包含一个 <h1>-<h6> 作为子元素
- `<body>`
- `<footer>`:代表最近的父级区块内容的页脚，用来展示作者、版权、相关文档链接
- `<header>`:描述最近的父级区块，一组介绍性描述或导航信息。通常包含h1-h6
- `<h1>...<h6>`
- `<hgroup>`
- `<nav>`
- `<section>`:通常包含h1-h6


## 文本内容

- `dl-dt-dd`:表示键值对，名词和解释
- `ul-ol-li`
- `<pre>`:表示预格式化(已排版)的内容
- `<p>`
- `<hr>`
- `<main>`:文档的主内容
- `<figure>`:自包含内容，通常带有标题 figcaption 。通常是插图、图表、照片、代码
- `<figcaption>`:通常是图表标题、图例、代码说明
- `<div>`


## inline text semantics

- `<a>`
- `<br>`
- `<time>`
- `<abbr>`:缩写。其title属性的含义为所缩写的全称
- `<dfn>`:展示术语的定义。最近的父级块必须包含dfn元素指定的术语的定义。

- `<cite>`:引述元素（Citation Element）。引述的作品（书、文章、电影、歌曲、节目等）的标题。
- `<q>`: quote element。cite属性表示来源URL。短引用使用q，不需要分段。长引用使用<blockquote>。

- `<code>`
- `<kbd>`: Keyboard Input Element 。用户输入内容/按键
- `<samp>`:计算机输出

- `<i>`:用于技术名词 / 外文短语 / 虚构人物
- `<em>`:表示侧重点的强调
- `<b>`:表示某种需要引起注意却又没有其他额外语义的内容。用于：摘要中的关键词 / 评介中的产品名称 / 文章的开篇内容 
- `<strong>`:表示内容的重要性

- `<small>`:用于：免责声明 / 许可证声明 / 注意事项
- `<u>`:表示用非文本进行的标注的内容。中文专名 / 拼写检查的错误内容
- `<s>`:表示不再准确或不再相关的内容
- `<mark>`:在引用的文字中使用，表示在当前文档中需要引起注意但原文中并没有强调的含义 (eg. 对一篇文章的分析中对原文的标注);表示与用户当前的行为相关的内容 (eg. 高亮显示搜索关键词)

- `<sub><sup>`: Subscript Element && Superscript Element 
- `<ruby><rp><rt><rtc>`:注音标识
- `<var>`:表示数学表达式或编程上下文中的变量
- `<wbr>`:Word Break Opportunity。

- `<bdi><bdo>`
- `<data>`
- `<span>`

## 图片和多媒体

- `<img>`
- `<picture>`
- `<map><area>`:图片热点
- `<audio>`:音频
- `<video>`:视频
- `<track>`:为多媒体元素指定文本轨。格式为.vvt 文件 [https://developer.mozilla.org/en-US/docs/Web/API/Web_Video_Text_Tracks_Format](https://developer.mozilla.org/en-US/docs/Web/API/Web_Video_Text_Tracks_Format)
- `<source>`:表示所在多媒体元素的可替代资源


## 嵌入内容

- `<embed>`:外部应用或可交互内容的整合入口
- `<iframe>`
- `<object>`:通用外部资源
- `<param>`:为 object 元素传递的参数


## 脚本

- `<canvas>`
- `<noscript>`
- `<script>`

## 标记更改
表示对当前文档内容进行的增添与删改,用来记录文档的编辑历史:

cite 属性指向对某个修改的说明文档的 URL

datetime 属性表示了修改发生的时间 (取值规范)


- `<del>`
- `<ins>`: Inserted Text


## 表格

- `<table>`
- `<thead>`
- `<tbody>`
- `<tfoot>`
- `<caption>`:表示所处的 table 的标题，通常是表格的第一个子元素。
- `<th>`
- `<tr>`
- `<td>`
- `<col>`
- `<colgroup>`


## 表单

- `<form>`
- `<label>`:可以和其它控件关联起来，通过for 属性或是将其它控件包裹起来
- `<button>`
- `<input>`
- `<textarea>`

- `<select>`
- `<optgroup>`
- `<option>`
- `<datalist>`
- `<fieldset>`:分组控件
- `<legend>`:fieldset 的标题

- `<output>`
- `<progress>`
- `<meter>`
## interactive elements

- `<details>`
- `<dialog>`
- `<menu>`
- `<menuitem>`
- `<summary>`

## Web Components

- `<content>`
- `<element>`
- `<shadow>`
- `<template>`

资源：

- [http://www.w3.org/TR/components-intro/](http://www.w3.org/TR/components-intro/)

## 废弃元素

参考链接：

- [https://developer.mozilla.org/en-US/docs/Web/HTML/Element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element)