---
layout: post
category : lessons
title: "node.js packages 学习笔记 -- glob"
tagline: "Supporting tagline"
tags : [node.js, glob]
---

项目地址：[https://github.com/isaacs/node-glob](https://github.com/isaacs/node-glob)

> glob functionality for node.js. Match files using the patterns the shell uses, like stars and stuff.

它使用 [minimatch](https://github.com/isaacs/minimatch) 库来做匹配。
> minimatch: A minimal matching utility. It works by converting glob expressions into JavaScript RegExp objects.

## 基础的
在解析路径之前，braced sections 会展开成一个集合。braced sections 是指以 `{` 开始，以 `}` 结束，里面包含任意数量的逗号分隔部分。

- `*` 在一个 path portion 匹配一个或多个字符
- `?` 匹配一个字符
- `[…]` 匹配字符区间，类似于正则区间。如果区间的第一个字符是 ! 或是 ^，则表示不匹配区间中任何字符。
- `!(pattern|pattern|pattern)`
- `?(pattern|pattern|pattern)` 匹配零次或一次
- `+(pattern|pattern|pattern)` 匹配一次或多次
- `*(a|b|c)` 匹配零次或多次
- `@(pattern|pat*|pat?erN)`
- `**` If a "globstar" is alone in a path portion, then it matches zero or more directories and subdirectories searching for matches. It does not crawl symlinked directories.

## Dots
以 `.` 开头的文件，不匹配任何 glob 模式，除非模式也是以 `.` 开头。

## basename 匹配
如果设置 `matchBase:true`，那么 `*.js` 会匹配 `test/simple/basic.js`

## 空集

## 方法

- `glob.hasMagic(pattern, [options])` 如果 pattern 中包含特殊字符，则返回 `true`，否则返回 `false`
- `glob.hasMagic(pattern, [options])` 执行异步 glob search.
- `glob.sync(pattern, [options])` 执行同步 glob search。返回匹配 pattern 的文件名数组。

## 类
### 属性
### 事件
### 方法
### 选项

## 与其它 fnmatch/glob 实现的对比
## windows
## 竞态
Glob searching, by its very nature, is susceptible to race conditions, since it relies on directory walking and such.

参考资料：

- [https://en.wikipedia.org/wiki/Glob_(programming)](https://en.wikipedia.org/wiki/Glob_(programming))
