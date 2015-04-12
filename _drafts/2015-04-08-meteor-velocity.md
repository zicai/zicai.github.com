---
layout: post
category : lessons
title: "meteor velocity"
tagline: "Supporting tagline"
tags : [meteor,test]
---

velocity 是Meteor的测试框架。

##测试框架
###Mocha
客户端和服务端的集成测试

```
meteor add mike:mocha
```

###jasmine
客户端集成测试，服务端单元测试

```
meteor add sanjo:jasmine
meteor add velocity:html-reporter
```

###nightwatch
浏览器验收测试。模拟用户交互。

```
meteor add velocity:nightwatch-framework
meteor add velocity:html-reporter
```
运行起来比较慢。


###cucumber
验收测试

```
meteor add xolvio:cucumber
```

###RobotFramework
端到端验收测试
##reporters
###html reporter

##7种测试模式
[http://www.meteortesting.com/blog/e72fe/the-7-testing-modes-of-meteor](http://www.meteortesting.com/blog/e72fe/the-7-testing-modes-of-meteor)
###APP-level
####APP-level 端到端
- mode#1
- mdoe#2
####APP-level 客户端集成测试
- mode#3
####APP-level 服务端集成测试
- mode#4
####APP-level 服务端单元测试
- mode#5
####APP-level 客户端单元测试
- mode#6

###Package-level 
