---
layout: post
category : Mobile
title : "Yeoman generator mobile 简介"
tagline: "Supporting tagline"
tags : []
---


##功能
这些功能都是可选的

- 支持[Bootstrap 3](http://getbootstrap.com/)，[TopCoat](http://topcoat.io/) , [Zurb Foundation](http://foundation.zurb.com/) 和[Pure](http://purecss.io/)
- 通过`srcset`生成响应式图片
- 生成你网站在不同viewport下的屏幕截图
- 使用 BrowserStack 作为云测试
- 引入[FastClick](https://github.com/ftlabs/fastclick) 避免 IOS touch 延迟
- 引入boilerplate作为全屏API
- 只包括你项目中用到的Modernizr功能
- 转换图片为WebP
- Includes a polyfill for [async. localStorage](https://github.com/slightlyoff/async-local-storage)

##开始

- 安装： `npm install -g generator-mobile`
- 运行： `yo mobile`
- 执行build： `grunt` 
- 运行本地server： `grunt serve`
- 屏幕截图：`grunt screenshots`
- 在BrowserStack里运行：`grunt open [nexus4|nexus 7|iphone 5]`

##为什么需要一个mobile web-app generator
mobile web 开发需求越来越多，但是开发一个mobile web需要做很多碎片化工作。我们想看看通过grunt如何简化开发过程。

针对mobile web开发中诸多痛点，我们包含了很多解决方案，例如：跨终端测试，目标设备屏幕快照，代码优化等等。

##移动优先的框架
[Bootstrap 3](http://getbootstrap.com/)，[TopCoat](http://topcoat.io/) , [Zurb Foundation](http://foundation.zurb.com/) 和[Pure](http://purecss.io/)都是很好的选择。

当你执行 `yo mobile` 时，你可以选择其中之一


## Grunt tasks
- [grunt-responsive-images](https://npmjs.org/package/grunt-responsive-images) 根据预定义的宽度生成多种分辨率的图片。适用于`srcset`或其他响应式图片方案，例如[Imager.js](https://github.com/BBC-News/Imager.js/)
- [grunt-autoshot](https://npmjs.org/package/grunt-autoshot)用来给你的网站在不同的viewport下生成屏幕快照
- [grunt-modernizr](https://npmjs.org/package/grunt-modernizr)用来生成精简的Modernizr 基于检测你网站用到的功能
- [grunt-svgmin](https://npmjs.org/package/grunt-svgmin)用来压缩你的SVG 文件
- [grunt-contrib-imagemin](https://npmjs.org/package/grunt-contrib-imagemin)用来优化图片大小
- [grunt-open](https://npmjs.org/package/grunt-open)用来打开浏览器窗口以指定设备、浏览器设置打开browserStack
- [grunt-webp](https://npmjs.org/package/grunt-webp)用来把图片编码成webp格式
- [grunt-concurrent](https://npmjs.org/package/grunt-concurrent)用来并行运行grunt tast
- [grunt-notify](https://npmjs.org/package/grunt-notify)用来桌面提醒当Grunt 出错时

## 建议
虽然我们决定不把下面的包含在generator-mobile workflow中，但是，你可能会用到他们

- [grunt-pagespeed](https://github.com/jrcryer/grunt-pagespeed) 使用Google PageSpeed服务运行一系列速度测试
- [grunt-montage](https://github.com/globaldev/grunt-montage) 和 [grunt-spritesheet](https://github.com/nicholasstephan/grunt-spritesheet) 
- [grunt-zopfli](https://github.com/mathiasbynens/grunt-zopfli) 使用Zopfli 压缩模式 压缩文件
- [grunt-stripmq](https://github.com/jtangelder/grunt-stripmq) generates fallback versions of mobile-first stylesheets
- [grunt-manifest](https://github.com/gunta/grunt-manifest) generates appcache manifests

> [github 地址](https://github.com/yeoman/generator-mobile) 