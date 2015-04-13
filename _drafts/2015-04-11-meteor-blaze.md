---
layout: post
category : lessons
title: "meteor blaze"
tagline: "Supporting tagline"
tags : [meteor]
---

Blaze 是一个非常强大的库，利用reactive HTML模板来穿件用户界面。使用Blaze，你的APP就无需在写更新逻辑。

模板标签集合了tracker的 transparent reactivity 和miniMongo的 数据库游标，这样DOM就会自动更新。

##细节
Blaze主要包含两部分：
- 一个模板编译器，用来将模板文件编译成javascript对象，
- 一个reactive DOM 引擎，在运行时构建和管理DOM。

##理念
- 平缓的学习曲线
- transparent reactivity。底层，Blaze使用tracker 自动跟踪合适重新计算模板helper。
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
- 


















