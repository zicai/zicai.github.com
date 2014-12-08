---
layout: post
category : 
title: "如何用Modern.IE在本地测试你的网站"
tagline: ""
tags : [自动化测试]
---

市面上有很多用来测试前端代码质量的工具。比如说JSHint和JSLint用来检查我们的JS文件，[W3C Markup validator](http://validator.w3.org/)用来检测我们的HTML代码是否有效，是否符合规范，[W3C CSS validator](http://jigsaw.w3.org/css-validator/)用来检测我们的样式表。其实，还有更多工具可用。

今天来介绍微软的[Modern.IE](http://modern.ie/)。它可以扫描你的网站，从中找出常见的代码问题并生成一个报告（可以做成pdf）。这份报告包含了每个测试的结果以及一些有关如何解决问题或提高代码质量的建议。用这种方式，你可以保证自己的代码符合当前最佳实践并且性能良好。而你只需提供目标网页URL即可。

##什么是Modern.IE

Modern.IE是一项服务，它提供了一系列不同的工具，可以从不同的角度和目标来测试我们的网站。例如：Modern.IE提供了几个免费的windows虚拟机用来在windows，Mac，或者Linux上运行任何版本的IE。

另一个功能就是由[BrowserStack](http://www.browserstack.com/)提供的免费的自动屏幕快照。这个工具可以在几分钟之内生成指定网站在一系列移动设备和桌面设备上显示效果的快照。这意味着你可以得到你的网站在Android系统上的浏览器，windows 8上的火狐、Opera，甚至那些不容易获取的设备，比如使用Safari的iPhone 4s上显示效果的快照。

接下来我们将深入探索如何扫描本地网站。

##如何分析一个本地网站

###安装扫描工具

Modern.IE用来扫描网站的工具可以在[github上获取](https://github.com/InternetExplorer/modern.IE-static-code-scan/)。在命令行中执行下面的命令，获取源代码拷贝

    git clone https://github.com/InternetExplorer/modern.IE-static-code-scan.git
    
或者直接在github上点击下载按钮，如下图：

![直接在github上点击下载按钮](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2014/08/1407913254download-source-button.png)

下载完成之后，你需要下载安装Node.js(版本0.10或更高版本)。安装Node.js之后，切换到扫描工具的源代码目录，执行下面的命令，安装所需依赖：

    npm install
    
最后一步，启动扫描服务，执行下面的命令：

    node app.js
    
在执行完上个命令后，你将会看到一条关于服务状态和使用端口的消息（端口默认为1337）。打开浏览器，访问http://localhost:[PORT-NUMBER]/ ，如果你没有修改默认设置，那么[PORT-NUMBER]为1337。

按照上面的步骤操作下来，如果一切正常的话，你将会看到下面的页面

![](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2014/08/1407913396offline-scan-tool.png) 

现在你就可以开始扫描你的本地网站了

###创建一个报告

一切准备妥当之后，你就可以开始扫描本地网站了。在开始之前，要注意一点就是当前版本的扫描工具依赖jQuery，Microsoft选用了jQuery CDN。也就是说你必须联网，即使你测试的是离线网站，否则扫描工具会报错（显示错误“Uncaught ReferenceError: $ is not defined”，因为它无法加载jQuery）.

输入你要扫描的网址，点击Scan按钮，如下图：

![输入你要扫描的网址，点击Scan按钮](http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2014/08/1407913422scanning-an-offline-page.png)

如果你正在使用一个需要验证的系统，例如HTTP Basic 和Digest，你可以指定用户名和密码。

扫描完成之后，工具会生成JSON格式的报告。

###JSON报告
扫描完成之后，工具会把扫描结果输出为JSON格式。一个成功的测试输出结果如下：

    “imageCompression”: {
        “testName”: “imageCompression”,
        “passed”: true
    }

一个失败的测试输出结果如下：

    “ie11tiles”: {
    	“testName”: “ie11tiles”,
    	“passed”: false,
    	“data”: {
    		“square70”: false,
    		“square150”: false,
    		“wide310”: false,
    		“square310”: false
    		“notifications”: false
    	}
    }
    
你可以选择自己写脚本转换测试报告，也可以在第二步的时候，通过点击Create Report按钮，将JSON格式的报告发送到Modern.IE 。如果你选择第二种方式，Modern.IE会显示本地测试报告，好像你在测试在线网站一样。需要注意的是截止本文发布，离线版本的工具因为一个问题导致不能在Modern.IE 上显示本地测试报告。

##结语

为了检测兼容性问题和性能提升，Modern.IE提供了许多用来分析网站的工具，不论是离线还是在线。感谢这个本地版本，它可以在网站上线之前进行测试，使我们可以避免在用户和客户之前出错。

你试过Modern.IE 了么？

原文地址： [http://www.sitepoint.com/test-site-locally-modern-ie/](http://www.sitepoint.com/test-site-locally-modern-ie/)