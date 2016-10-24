---
layout: post
category : test
title: "Phantomjs 基础"
tagline: "Supporting tagline"
tags : [phantomjs]
---

##phantomjs 是什么？
PhantomJS是一个无界面的,可脚本编程的WebKit浏览器引擎。它原生支持多种web 标准：DOM 操作，CSS选择器，JSON，Canvas 以及SVG。

##phantomjs 可以做什么？
- 无UI界面的网站测试
- 屏幕快照
- 页面操作自动化
- 网络监控

##下载安装

找到对应版本[下载](https://bitbucket.org/ariya/phantomjs/downloads)安装，或是通过源码编译安装。

###window
下载`phantomjs-1.9.8-windows.zip`并解压，`phantomjs.exe`直接可用。方便起见，把`phantomjs.exe`所在目录加入到`PATH`中

###MAC OS X
下载`phantomjs-1.9.8-macosx.zip`并解压，`bin/phantomjs`直接可用。

或者通过Homebrew安装

    brew update && brew install phantomjs

或者通过MacPorts安装

    sudo port selfupdate && sudo port install phantomjs

###Linux
64位下载`phantomjs-1.9.8-linux-x86_64.tar.bz2`

32位下载`phantomjs-1.9.8-linux-i686.tar.bz2`

安装渲染所需（[FreeType](http://www.freetype.org/),[Fontconfig](http://www.freedesktop.org/wiki/Software/fontconfig)）依赖：

    sudo apt-get install fontconfig freetype libfreetype.so.6 libfontconfig.so.1 libstdc++.so.6
    
 

##快速开始
安装完phantomjs 之后，将其可执行路径加入到PATH中。

###hello world
新建`hello.js`，添加如下代码：

    console.log('Hello, world!');
    phantom.exit();

在命令行里执行：

    phantomjs hello.js

会输出：

    Hello, world!

第一行代码，`console.log`会打印字符串到命令行。第二行代码，`phantom.exit`会终止执行过程。

注意：一定不要忘了在代码里调用`phantom.exit`，否则PhantomJS将不会被终止

###页面加载

通过`page` 对象，我们可以加载，分析，渲染页面。

下面的代码展示了`page` 对象最简单的用法。它加载example.com然后将其保存为一个图片`example.png`。

    var page = require('webpage').create();
    page.open('http://example.com', function() {
      page.render('example.png');
      phantom.exit();
    });

PhantomJS基于Webkit引擎，可以布局和渲染页面，我们可以利用这一点给网页拍照。

下面的`loadspeed.js`,加载一个指定的URL，然后测量加载页面所需时间。

    var page = require('webpage').create(),
      system = require('system'),
      t, address;
    
    if (system.args.length === 1) {
      console.log('Usage: loadspeed.js <some URL>');
      phantom.exit();
    }
    
    t = Date.now();
    address = system.args[1];
    page.open(address, function(status) {
      if (status !== 'success') {
        console.log('FAIL to load the address');
      } else {
        t = Date.now() - t;
        console.log('Loading ' + system.args[1]);
        console.log('Loading time ' + t + ' msec');
      }
      phantom.exit();
    });
    
从命令行执行(注意有`http`)：

    phantomjs loadspeed.js http://www.google.com
    
输出结果类似下面这样：

Loading http://www.google.com Loading time 719 msec

###执行脚本

想要在页面的上下文里执行javascript或者coffeescript代码，要使用`evaluate()`函数。`evaluate()`可以返回一个对象，但必须是一个简单的对象，不能包含函数或者闭包。

下面的代码展示了如何显示一个网页的title:

    var page = require('webpage').create();
    page.open(url, function(status) {
      var title = page.evaluate(function() {
        return document.title;
      });
      console.log('Page title is ' + title);
      phantom.exit();
    });
    
默认情况下，页面代码里的`console`输出的消息，以及`evaluate()`里的`console`输出的消息都不会显示出来。不过，通过使用`onConsoleMessage`回调函数,你可以改写这一行为，上面的实例可以改为：

    var page = require('webpage').create();
    page.onConsoleMessage = function(msg) {
      console.log('Page title is ' + msg);
    };
    page.open(url, function(status) {
      page.evaluate(function() {
        console.log(document.title);
      });
      phantom.exit();
    });

`evaluate()`里的代码就像在浏览器里执行一样，所以像标准的DOM操作、CSS选择等都可以正常进行。我们可以利用这一点将一些页面工作自动化。

###网络请求与返回

当页面向服务器请求资源时，通过onResourceRequested 和 onResourceReceived 回调函数可以 记录request和response。例如：

    var page = require('webpage').create();
    page.onResourceRequested = function(request) {
      console.log('Request ' + JSON.stringify(request, undefined, 4));
    };
    page.onResourceReceived = function(response) {
      console.log('Receive ' + JSON.stringify(response, undefined, 4));
    };
    page.open(url);
    
基于此我们可以进行一些页面性能的分析。