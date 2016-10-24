---
layout: post
category : test
title: "Karma 入门"
tagline: "Supporting tagline"
tags : [karma]
---
##是什么
karma是一个简单的工具，使用它可以在多个真实的浏览器中运行javascript代码
> karma的主要用途是使你的TDD开发简单、快速、有乐趣。

##什么时候应该使用karma

- 想在真实的浏览器中测试代码
- 想在多个浏览器中测试代码(桌面、手机、平板等)
- 在开发过程中运行测试
- 在CI服务器上运行测试
- 每次保存时运行测试
- 想使用Lstanbul 生成测试报告
- 想在源文件中使用requireJS

##安装
Karma 构建在Node.js 之上，以npm包形式安装
###安装Node.js
###安装Karma和插件
建议到项目目录中，局部安装Karma及插件

    # 安装 Karma:
    $ npm install karma --save-dev
    # 安装项目所需插件:
    $ npm install karma-jasmine karma-chrome-launcher --save-dev

上面的命令会安装`karma`, `karma-jasmine` 和 `karma-chrome-launcher`包到你当前工作目录中node_modules文件夹中，并把这些包名保存到package.json 里的`devDependencies` 。这样该项目的其他开发者只需运行`npm install` 就可以安装这些依赖。

    # 运行 Karma:
    $ ./node_modules/karma/bin/karma start

###命令行接口
每次运行karma都需要输入`./node_modules/karma/bin/karma start`太逊了，所以全局安装`karma-cli`会很有用

    $ npm install -g karma-cli
    
接下来你只需输入`karma`就可以随处运行karma了

##配置
开始测试之前，karma需要了解你的项目，所以需要一个配置文件。

###生成配置文件
使用`karma init`生成配置文件

    $ karma init my.conf.js
    
    Which testing framework do you want to use ?
    Press tab to list possible options. Enter to move to the next question.
    > jasmine
    
    Do you want to use Require.js ?
    This will add Require.js plugin.
    Press tab to list possible options. Enter to move to the next question.
    > no
    
    Do you want to capture a browser automatically ?
    Press tab to list possible options. Enter empty string to move to the next question.
    > Chrome
    > Firefox
    >
    
    What is the location of your source and test files ?
    You can use glob patterns, eg. "js/*.js" or "test/**/*Spec.js".
    Enter empty string to move to the next question.
    > *.js
    > test/**/*.js
    >
    
    Should any of the files included by the previous patterns be excluded ?
    You can use glob patterns, eg. "**/*.swp".
    Enter empty string to move to the next question.
    >
    
    Do you want Karma to watch all the files and run the tests on change ?
    Press tab to list possible options.
    > yes
    
    Config file generated at "/Users/vojta/Code/karma/my.conf.js".
    
配置文件可以用CoffeeScript来写。事实上，如果你运行`karma init`，后面跟着一个`*.coffee`扩展名，例如：`karma init karma.conf.coffee`,就会生成一个CoffeeScript文件。

当然你也可以手写配置文件，或从其他项目拷贝配置文件。

###启动karma
当启动Karma时，配置文件的路径可以作为第一个参数

默认情况下，karma会在当前目录中寻找`karma.conf.js` 或者 `karma.conf.coffee`

    # Start Karma using your configuration:
    $ karma start my.conf.js
    
配置文件更详细说明参见[configuration file docs]()

###命令行参数
当执行karma时，配置文件中已有的一些配置项，可以被命令行参数重写

    karma start my.conf.js --log-level debug --single-run
    
如果您想知道都有哪些可用选项，运行`karma start --help`

###集成到Grunt 或 Gulp
- [grunt-karma](https://github.com/karma-runner/grunt-karma)
- [gulp-karma](https://github.com/karma-runner/gulp-karma)

##工作原理
从本质上来说，karma是一个工具，用来构建一个web server ，web server 会到每一个连接上的浏览器中执行测试代码。每一个浏览器的测试结果都会通过命令行展示给开发人员，这样他们就可以看到哪个浏览器通过或失败。

有两种捕获浏览器的方式：

- 手动的，通过访问Karma服务器监听的URL
- 自动的，让karma知道在启动的时候，需要寻找那些浏览器 参见[browsers]()

同时，Karma会监测配置文件中指定的所有文件，当其中任何一个文件发生变化时，就会发送一个信号到testing server,server 会通知所有连接的浏览器 重新执行测试。接下来，每个浏览器在一个Iframe中重新加载源代码，执行测试，最后将结果返回给server。

server收集浏览器返回的结果，呈献给开发者。

这只是一个非常简单的介绍，如果你只是使用karma，完全没必要了解karma内部细节。但是，如果你非常感兴趣karma的设计与实现，可以读读[这篇文章](https://github.com/karma-runner/karma/raw/master/thesis.pdf)

##FAQ 

- 我可以结合karma和其他测试框架么？


    可以的，对于常用的测试框架（例如 Jasmine，Mocha，Qunit）都有插件。
-  可以用karma来做端到端测试（end to end testing）么？


    Karma主要设计用来做低层级的测试（单元测试）。如果你想做高层级测试，我们推荐使用[protractor](https://github.com/angular/protractor)
- 我能在CI服务器上使用Karma么？


    当然，你可以看看相关文档[Jenkins](http://karma-runner.github.io/0.12/plus/jenkins.html),[Travis](http://karma-runner.github.io/0.12/plus/travis.html),[Semaphore](http://karma-runner.github.io/0.12/plus/semaphore.html)
- 我该使用哪个版本的Karma


    npm上最新稳定版（`npm install karma`）
- Karma运行在哪个版本的Node.js


    最新的两个稳定版。当前也就是0.8和0.10。