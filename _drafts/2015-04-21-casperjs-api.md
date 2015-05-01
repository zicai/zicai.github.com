---
layout: post
category : lessons
title: "casperjs API"
tagline: "Supporting tagline"
tags : [casperjs]
---

API分为四个模块

- casper
- clientutils
- colorizer
- mouse
- tester
- utils

##casper
###casper类
新建casper实例最简单的方法是：

```
var casper = require('casper').create();
```

create()方法可以接受options 参数：

```
var casper = require('casper').create({
    verbose: true,
    logLevel: "debug"
});
```

###Casper.options
可以把option对象传给构造函数：

```
var casper = require('casper').create({
    clientScripts:  [
        'includes/jquery.js',      // These two scripts will be injected in remote
        'includes/underscore.js'   // DOM on every request
    ],
    pageSettings: {
        loadImages:  false,        // The WebPage instance used by Casper will
        loadPlugins: false         // use these settings
    },
    logLevel: "info",              // Only "info" level messages will be logged
    verbose: true                  // log messages will be printed out to the console
});
```

也可以再运行时修改配置：

```
var casper = require('casper').create();
casper.options.waitTimeout = 1000;
```

###Casper prototype

导航相关

- start()
- then()
- bypass()
- repeat()
- run()
- thenBypass()
- thenBypassIf()
- thenBypassUnless()
- thenClick()
- thenEvaluate()
- thenOpen()
- thenOpenAndEvaluate()
- eachThen()

获取页面内容

- getCurrentUrl()
- getElementAttribute()
- getElementsAttribute()
- getElementBounds()
- getElementsBounds()
- getElementInfo()
- getElementsInfo()
- getFormValues()
- getGlobal()
- getHTML()
- getPageContent()
- getTitle()
- fetchText()
- withFrame()
- withPopup()
- visible()

表单相关

- fill()
- fillSelector()
- fillLables()
- fillXpath()

请求相关

- open()
- setHttpAuth()
- download()

浏览器相关

- clear()
- die()
- evaluate()
- evaluateOrDie()
- exit()
- userAgent()
- viewport()
- zoom()
- back()
- forward()
- reload()


等待

- unwait()
- wait()
- waitFor()
- waitForAlert()
- waitForPopup()
- waitForResource()
- waitForUrl()
- waitForSelector()
- waitWhileSelecotr()
- waitForSelectorTextChange()
- waitForText()
- waitUntilVisible()
- waitWhileVisible()



鼠标键盘相关：

- mouseEvent()
- scrollTo()
- scrollToBottom()
- click()
- clickLable()
- sendKeys()


调试相关：

- debugHTML()
- debugPage()
- exists()
- resourceExists()
- log()
- echo()
- warn()
- status()

截图相关：

- capture()
- captureBase64()
- captureSelector()

辅助函数：

- base64encode()
- toString()
- each()

##clientutils
client-side utilities用来注入到Remote DOM环境中，
##colorizer
##mouse
Mouse 类是各种鼠标操作(move,click,double-click,rollovers)的抽象。它依赖一个Casper实例，来获取DOM。用下面的方式创建：

```
var casper = require("casper").create();
var mouse = require("mouse").create(casper);
```

注意：
casper实例中已经定义了一个mouse属性，所以通常不需要自己创建。例如：

```
casper.then(function() {
    this.mouse.click(400, 300); // clicks at coordinates x=400; y=300
});
```
- click()
- doubleclick()
- rightclick()
- down()  触发mousedown事件
- move()
- up()
##tester
默认，你可以通过任何casper实例的test属性来获取一个Tester类实例。
##utils
提供简单地辅助函数

###使用

```
var utils = require('utils');

utils.dump({plop: 42});
```

