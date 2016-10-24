---
layout: post
category : test
title: "Karma 浏览器设置"
tagline: "Supporting tagline"
tags : [karma]
---

手动捕获浏览器费时费力，所以karma可以代替你做这件事，你只需把要使用的浏览器添加到配置文件，例如：

    browsers: ['Chrome']
    
接下来，karma就可以自动的捕获这些浏览器，关闭的时候也一样

需要注意的是：大部分浏览器启动器需要额外的插件

##可用的浏览器启动器

- [Chrome and Chrome Canary](https://github.com/karma-runner/karma-chrome-launcher)
- [Firefox](https://github.com/karma-runner/karma-firefox-launcher)
- [Safari](https://github.com/karma-runner/karma-safari-launcher)
- [PhantomJS](https://github.com/karma-runner/karma-phantomjs-launcher)
- [Opera](https://github.com/karma-runner/karma-opera-launcher)
- [IE](https://github.com/karma-runner/karma-ie-launcher)
- [SauceLabs](https://github.com/karma-runner/karma-sauce-launcher)
- [BrowserStack](https://github.com/karma-runner/karma-browserstack-launcher)
- [更多](https://www.npmjs.org/browse/keyword/karma-launcher)

下面的例子说明了如何添加Firefox到测试中：

    # 首先通过npm安装相应启动器
    $ npm install karma-firefox-launcher --save-dev

然后添加到配置文件中：

    module.exports = function(config) {
      config.set({
        browsers : ['Chrome', 'Firefox']
      });
    };

##手动捕获任何浏览器

在任何浏览器中打开`http://<hostname>:<port>/`，`<hostname>`是karma server所在的测试机的IP，`<port>`是karma server 监听的端口（默认是9876），就可以捕获该浏览器。

这样你就可以捕获在同一局域网内任何浏览器，包括手机或平板上的浏览器。

##配置启动器

一些启动器可以有配置项：

    sauceLabs: {
      username: 'michael_jackson'
    }

或者定义为自定义启动器

    customLaunchers: {
      chrome_without_security: {
        base: 'Chrome',
        flags: ['--disable-web-security']
      },
      sauce_chrome_win: {
        base: 'SauceLabs',
        browserName: 'chrome',
        platform: 'windows'
      }
    }
    
##纠正浏览器位置
每个插件针对特定操作系统都会有一些默认的浏览器路径，你可以通过`<BROWSER>_BIN`环境变量修改这些设置，或者新建一个符号连接。

- POSIX shells

        # Changing the path to the Chrome binary
        $ export CHROME_BIN=/usr/local/bin/my-chrome-build
    
        # Changing the path to the Chrome Canary binary
        $ export CHROME_CANARY_BIN=/usr/local/bin/my-chrome-build
    
        # Changing the path to the PhantomJs binary
        $ export PHANTOMJS_BIN=$HOME/local/bin/phantomjs

- Windows cmd.exe

        C:> SET IE_BIN=C:\Program Files\Internet Explorer\iexplore.exe
        
- Windows Powershell

        $Env:FIREFOX_BIN = 'c:\Program Files (x86)\Mozilla Firefox 4.0 Beta 6\firefox.exe'


##Custom browsers