---
layout: post
category : scrapy
title: 前端自动化
tagline: "Supporting tagline"
tags : []
---

##自动化之前的工作场景
1. 新建项目A文件夹，再在A文件夹里创建html、css、js、images所需的各个文件夹
2. 将要用到的css文件(例如：reset.css，bootstrap等)，js文件(例如：Jquery，各种插件等)从以前的项目拷贝到当前项目中
3. 准备的差不多了，开始切图。写代码，浏览器刷新看效果，改代码，浏览器刷新看效果，再改代码，再刷新。。。。。
4. 如果在项目中用到 Less 或者 Sass，时不时的还需要将它们编译成css看效果
5. 需要用到新插件的话，google一下，找到下载，按照文档说明拷贝到对应目录
6. 切图完成之后。还要压缩css、js、图片，混淆js，单元测试等等。

这些重复性的劳动既耗费时间又没有技术含量。还好，借助一些自动化工具我们就可以跟它们说bye bye 了。

####总结上面的开发流程，主要是下面四点：
- 开发环境初始化
- 样式、脚本的依赖管理
- 文件编译、压缩合并、混淆
- 自动化测试 等等

##解决之道
通过一些很好用的自动化工具，我们可以将上面的各个部分都自动化，只需敲几个命令就可以走完整个流程，并及时得到运行结果的反馈。

####对应的自动化工具：
- 开发环境初始化-----------------[yeoman](http://yeoman.io/)
- 样式、脚本的依赖管理----------[bower](http://bower.io/)
- 文件编译、压缩合并、混淆-----[grunt](http://www.gruntjs.org/)及其插件
- 自动化测试 等等----------------[karma](http://karma-runner.github.io/)等

注意：上面针对每一部分只列举了一个自动化工具，其实还有很多替代选择。例如：你可以用[gulp](http://gulpjs.com/) 代替grunt

后面我们会以`yeoman + grunt + bower`来做示例。同时，由于现在前端开发中，已经很少直接写CSS文件了，一般都会用 Less 或者 Sass，所以下面假设项目使用Sass和[compass](http://compass-style.org/)。

##搭建前端自动化环境

###安装[Node.js](http://nodejs.org/)

默认安装NPM（Node Package Manager）并自动添加环境变量。
在命令行输入下面命令，检查安装是否成功

    node -v
    npm -v

###安装Yeoman、bower、grunt
    npm install -g yo bower grunt-cli
   使用Yeoman

    yo
    
进入提示界面，选择安装、更新或是运行某个generator。

走到这一步，`yeoman + grunt + bower`就都安装好了。你已经可以开始你的前端自动化之旅了。接下来是Sass相关的环境搭建，你可以根据实际情况选择。

Sass是一种"CSS预处理器"，可以让CSS的开发变得简单和可维护。Compass是Sass的工具库，Compass在Sass的基础上，封装了一系列有用的模块和模板，补充Sass的功能。Compass是用Ruby语言开发的，所以需要先安装ruby。

###安装Ruby

windows上安装Ruby推荐使用[RubyInstaller](http://rubyinstaller.org/)(需翻墙)
检查安装是否成功

    ruby -v
    
###安装Compass

    gem install compass
    
会自动安装Sass

##更多详细使用方法

###yeoman
常用命令

    npm install -g generator-angular #手动安装生成器
    yo angular                       #搭建AngularJS项目脚手架

###Grunt
参考[http://www.gruntjs.org/](http://www.gruntjs.org/)

常见自动化任务对应的插件：

- grunt-contrib-watch: 监视磁盘文件，一旦更改就会重新运行指定的任务，例如使http服务器重新加载源文件
- grunt-contrib-ugligy: 压缩混淆JS源文件
- grunt-contrib-clean: 用于清理指定文件（夹），一般是构建之前或之后进行
- grunt-contrib-coffee: 将CoffeeScript编译为JavaScript
- grunt-contrib-compass: 调用Compass工具生成CSS文件
- grunt-contrib-concat: 合并源文件
- grunt-contrib-copy: 复制文件（夹）
- grunt-contrib-handlebars: 将handlebars模板预编译为JST文件，提高运行时性能
- grunt-contrib-jasmine: 借助Jasmine在PhantomsJS中运行单元测试，结合grunt-template-jasmine-istanbul，还能实现单元测试覆盖率计算
- grunt-contrib-jshint: JS代码质量检查工具，类似jsLint


###bower

常用命令

    bower install <dep>..<depN>         #给项目安装一个或多个依赖
    bower uninstall angular-ui          #移除某个依赖
    bower update <dep>                  #将某个依赖更新到最新版本
    bower list                          #列出当前项目安装的依赖
    bower search <dep>                  #在Bower registry查找某个依赖




