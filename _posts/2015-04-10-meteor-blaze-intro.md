---
layout: post
category : meteor
title: "Meteor Blaze 简介"
tagline: "Supporting tagline"
tags : [blaze]
---

Blaze 是一个非常强大的库，利用 reactive HTML 模板来创建用户界面。使用Blaze，你的 APP 就无需再写更新逻辑。

模板标签集合了 tracker 的 transparent reactivity 和 miniMongo 的 数据库游标，这样 DOM 就会自动更新。

##细节
Blaze主要包含两部分：

- 一个模板编译器，用来将模板文件编译成 javascript 对象，并由 Blaze runtime library 来运行。Blaze还提供了一个 compiler 工具链(think LLVM) ,可以用来支持任意的模板语法。
- 一个 reactive DOM 引擎，在运行时构建和管理DOM，通过模板或是直接由 APP 调用，它的功能是 reactive 更新区域，列表，属性，事件代理，以及很多用于辅助开发者的回调函数和钩子。

Blaze 经常被拿来和 React、Angular、Ember、Polymer、Knockout 以及其它框架进行比较。Blaze 特别之处在于对开发者的不懈关注。

##理念
- 平缓的学习曲线
- transparent reactivity。Blaze 底层使用 tracker 自动跟踪合适重新计算模板helper。
- 干净的模板
- 可插件

##和其他库对比
- backbone
- ember
- angular polymer
- react

##未来
- 组件
- 表单
- 移动端 动画
- javascript表达式



参考：[https://atmospherejs.com/meteor/blaze](https://atmospherejs.com/meteor/blaze)













