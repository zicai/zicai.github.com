---
layout: post
category : 
title: "CasperJS 基础"
tagline: "Supporting tagline"
tags : [CasperJS]
---
##CasperJS是什么？

CasperJS是一个开源的导航脚本处理和测试工具，基于PhantomJS。CasperJS简化了页面间导航的定义过程，提供了处理常见任务的实用的高级函数、方法和语法糖。

##CasperJS可以做什么？

- 定义、排列页面间导航的步骤
- 表单的填充、提交
- 点击、跟踪超链接
- 区域、页面截图
- 测试远程DOM
- 记录事件
- 资源下载，包括二进制资源
- 编写功能测试套件，可以将结果以JUnit XML形式导出
- 抓取网页内容

##安装
CasperJS可以在Mac、windows和大多数Linux上使用。CasperJS可以结合PhantomJS(Webkit内核)使用，也可以结合SlimerJS(Gecko内核)使用。

###依赖
- PhantomJS 1.8.2及以后版本
- Python 2.6及以后版本

###通过Homebrew安装（OS X）
首先更新Formulaes:

    $ brew update

安装development版本(推荐):

    $ brew install casperjs --devel

或是安装1.0.x稳定版:

    $ brew install casperjs


如果你已经安装了CasperJS，可以用`upgrade`更新到最新版：

    $ brew upgrade casperjs


##通过npm安装

    $ npm install -g casperjs

   
注意：虽然可以通过npm安装CasperJS，但是它并不是一个NodeJS包，它也不能require NodeJS原生模块。


##通过git安装

从github上下载代码：

    $ git clone git://github.com/n1k0/casperjs.git
    $ cd casperjs
    $ ln -sf `pwd`/bin/casperjs /usr/local/bin/casperjs


PhantomJS和CasperJS安装完成之后，可以用下面方式检测是否安装成功：

    $ phantomjs --version
    1.9.2
    $ casperjs
    CasperJS version 1.1.0-DEV at /Users/niko/Sites/casperjs, using phantomjs version 1.9.2
    # ...

或者是SlimerJS:

    $ slimerjs --version
    Innophi SlimerJS 0.8pre, Copyright 2012-2013 Laurent Jouanneau & Innophi
    $ casperjs
    CasperJS version 1.1.0-DEV at /Users/niko/Sites/casperjs, using slimerjs version 0.8.0



##通过存档安装

也可以直接下载CasperJS代码存档：

**最新开发版(master branch):**

- https://github.com/n1k0/casperjs/zipball/master (zip)
- https://github.com/n1k0/casperjs/tarball/master (tar.gz)

**最新稳定版:**

- https://github.com/n1k0/casperjs/zipball/1.0.3 (zip)
- https://github.com/n1k0/casperjs/tarball/1.0.3 (tar.gz)

下载完成之后，剩下的操作和通过git安装一样



##windows上安装CasperJS

###Phantomjs 

- 将phantomjs安装路径加到环境变量`PATH`里，例如:`";C:\phantomjs"`


###CasperJS 

- 将casperjs安装路径加到环境变量`PATH`里，例如: `";C:\casperjs\bin"`
- If your computer uses both discrete and integrated graphics you need to disable autoselect and explicitly choose graphics processor - otherwise ``exit()`` will not exit casper.

安装完成后就可以用下面方式运行casper脚本：

    C:> casperjs myscript.js



##已知bug和缺陷

- 由于异步的原因，CasperJS 不能很好地兼容 [PhantomJS' REPL](http://code.google.com/p/phantomjs/wiki/InteractiveModeREPL) 




