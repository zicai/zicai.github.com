---
layout: post
category : test
title: "Karma 配置"
tagline: "Supporting tagline"
tags : [karma]
---

##概述
Karma配置文件以Node.js模块方式被加载。

    // karma.conf.js
    module.exports = function(config) {
      config.set({
        basePath: '../..',
        frameworks: ['jasmine'],
        //...
      });
    };
    # karma.conf.coffee
    module.exports = (config) ->
      config.set
        basePath: '../..'
        frameworks: ['jasmine']
        # ...

##File Patterns
所有需要指定文件路径的配置选项，都是用[minimatch](https://github.com/isaacs/minimatch)库，方便你轻松的列出需要包含和剔除的文件

例如：

- `**/*.js`: 所有子目录中以`js`结尾的文件
- `**/!(jquery).js`: 同上, 但是不包括 "jquery.js"
- `**/(foo|bar).js`: 所有子目录中, 所有 "foo.js" 或 "bar.js" 文件


##配置项

###autoWatch

类型：布尔值

默认值：true

命令行：--auto-watch, --no-auto-watch

描述：开启或关闭监测文件，当文件发生变化时自动执行测试
###autoWatchBatchDelay
类型：数字

默认值：250

描述：当开启autoWatch时，karma会尝试将多个修改打包到一起运行一次测试代码，以此来减少运行次数。该配置项告诉karma在发生更改后需要等待多长时间（毫秒）再开始执行测试。
###basePath
类型：字符串

默认值：''

描述：用来解析定义在`files` 和 `exclude`里的相对路径的根目录。如果`basePath`是一个相对路径，那么它会被解析为相对于配置文件的 `__dirname`
###browserDisconnectTimeout
类型：数字

默认值：2000

描述：karma需要等待多长时间来允许浏览器重新连接（毫秒）。

浏览器通常会断开连接，但实际上测试仍在正常运行。karma并不会把断开连接立即判定为失败，而是会等待browserDisconnectTimeout 毫秒，如果在这期间，浏览器重新连接上，那么一切正常。

###browserDisconnectTolerance
类型：数字

默认值：0

描述：允许断开连接的次数。

`disconnectTolerance`的值代表了当断开连接时，浏览器尝试重连的最大次数。通常任何断开都会被视为失败，但是该选项允许你定义一个容忍度，允许Karma server与浏览器断开连接再重连。

###browserNoActivityTimeout
类型：数字

默认值：10000

描述：karma在断开一个浏览器连接之前会等待多长时间（毫秒）来接收信息

在测试执行过程中，如果在browserNoActivityTimeout（毫秒）时间内，Karma没收到任何来自某浏览器的消息，那么它会断开与该浏览器的连接。

###browsers
类型：数组

默认值：[]

命令行：--browsers Chrome,Firefox

可用值：

- Chrome (启动器已经和karma一并安装)
- ChromeCanary (启动器已经和karma一并安装)
- PhantomJS (启动器已经和karma一并安装)
- Firefox (启动器需要karma-firefox-launcher 插件)
- Opera (启动器需要karma-opera-launcher  插件)
- Internet Explorer (启动器需要karma-ie-launcher  插件)
- Safari (启动器需要karma-safari-launcher  插件)

描述：需要启动和控制的浏览器列表。当karma启动时，它会同时启动列表里的浏览器。当karma关闭时，它也会关闭列表里的浏览器。你可以通过访问Karma web server监听的url，手动的让karma控制任何浏览器。

更多信息参见[config/browsers](http://karma-runner.github.io/0.12/config/browsers.html)

其他的启动器可以通过[插件](http://karma-runner.github.io/0.12/config/plugins.html)定义
###captureTimeout
类型：数字

默认值：60000

描述：捕获浏览器的过期时间（毫秒）

`captureTimeout`代表了允许浏览器启动并连上karma的最大时间。在该时间内，如果任何一个浏览器没有连接上，karma会断开它，并尝试重新连接，如果尝试三次都没有成功，karma就会放弃。

###client.args
类型：数组

默认值：undefined

命令行：当使用`karma run` 时，`--`后所有参数

描述：当通过命令行提供了额外的参数给`karma run` 时，这些参数会通过`karma.config.args`传递给test adapter。

###colors
类型：布尔值

默认值：true

命令行：--colors, --no-colors

描述：开启或关闭彩色输出
###exclude
类型：数组

默认值：[]

描述：不会被加载的 文件/模式 列表
###files
类型：数组

默认值：[]

描述：被加载到浏览器的 文件/模式 列表
###frameworks
类型：数组

默认值：[]

描述：你要使用的测试框架列表，通常，你会设定为['jasmine'], ['mocha'] 或 ['qunit']

请注意：所有的框架都需要额外的插件/框架库 (通过npm安装)
参见[plugins](http://karma-runner.github.io/0.12/config/plugins.html)
###hostname
类型：字符串

默认值：'localhost'

描述：当捕获到浏览器时用到的hostname
###logLevel
类型：常量

默认值：config.LOG_INFO

命令行：--log-level debug

可用值：

- config.LOG_DISABLE
- config.LOG_ERROR
- config.LOG_WARN
- config.LOG_INFO
- config.LOG_DEBUG

描述：日志记录级别
###loggers
类型：数组

默认值：[{type: 'console'}]

描述：日志输出器列表。可以使用[log4js](https://github.com/nomiddlename/log4js-node)
###plugins
类型：数组

默认值：['karma-*']

描述：需要加载的插件列表。插件可以是一个字符串，或者是一个inline plugin-Object。默认情况下，karma加载所有以karma-*开始的兄弟npm 模块
###port
类型：数字

默认值：9876

命令行：--port 9876

描述：web server 监听的端口
###preprocessors
类型：对象

默认值：{'**/*.coffee': 'coffee'}

描述：用到的预处理器映射表
###proxies
类型：对象

默认值：{}

描述：代理映射表

例如：

    proxies: {
    '/static': 'http://gstatic.com',
    '/web': 'http://localhost:9000'
    },

###proxyValidateSSL
类型：布尔值

默认值：true

描述：当发现非法的SSL证书时，karma或者浏览器是否需要报错
###reportSlowerThan
类型：数字

默认值：0

描述：karma会报告所有慢于给定时间的测试，默认情况是禁用的，因为值为0.
###reporters
类型：数组

默认值：['progress']

命令行：--reporters progress,growl

可用值：

- dots
- progress


描述：reporters列表。其它的 reporters, 例如 growl, junit, teamcity 或者 coverage 可以以插件方式加载
###singleRun
类型：布尔值

默认值：false

命令行：--single-run, no-single-run

描述：CI模式。

如果设为`true`，karma会启动并捕获浏览器，运行测试然后退出，至于exit code 是0还是1，就要看是否所有测试都通过还是有任何测试失败。
###transports
类型：数组

默认值：['websocket', 'flashsocket', 'xhr-polling', 'jsonp-polling']

描述：浏览器与测试服务器之间允许的传输方式。该配置会传递给`socket.io`（用于管理浏览器与测试服务器的通信）。
###client.useIframe
类型：布尔值

默认值：true

命令行：在iframe里运行测试还是打开新窗口

描述：如果设为`true`，karma会在一个Iframe里运行测试。如果设为`false`，karma会在一个新窗口运行测试。因为一些测试可能无法再iframe里运行，需要打开新窗口。
###client.captureConsole
类型：布尔值

默认值：true

描述：捕获所有console输出，并输送到terminal
###urlRoot
类型：字符串

默认值：'/'

描述：karma运行的基础路径

所有karma url都会以urlRoot 做前缀。当使用代理时，这非常有用，因为有时候你会想代理一个karma已经占用的url。