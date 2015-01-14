---
layout: post
category : scrapy
title: scrapy 导读
tagline: "Supporting tagline"
tags : [scrapy]
---
原文地址：[http://doc.scrapy.org/en/latest/intro/tutorial.html](http://doc.scrapy.org/en/latest/intro/tutorial.html)
##Scrapy 一瞥

 - 定义你想要抓取的数据模型 (`scrapy.Item`)
 - 写蜘蛛(`CrawlSpider`)，
 - 运行蜘蛛 `scrapy crawl spidername`

##Scrapy 功能

 - 内置从HTML和XML中选择并提取数据的能力
 - 内置数据净化能力，通过一系列可重用的过滤器（item loaders）
 - 内置支持数据导出成多种格式(JSON,CSV,XML),且支持多种存储（FTP,S3,本地文件）
 - 一个媒体管道用来自动下载图片或其他媒体资源
 - 支持扩展“Support for extending Scrapy by plugging your own functionality using signals and a well-defined API (middlewares, extensions, and pipelines).”
 - 内置大量中间件和扩展，用来：
    - cookie和session 处理
    - HTTP压缩
    - HTTP验证
    - HTTP缓存
    - user-agent spoofing
    - robots.txt
    - 限制爬行深度
    - 等等
 - 强壮的编码支持，用于解决国外不标准和broken的编码声明
 - 支持基于预定义模板快速创建蜘蛛（genspider 命令）
 - 可扩展的状态收集，用来监控蜘蛛运行状况
 - Interactive shell console用来尝试xpath，和调试蜘蛛
 - “A System service designed to ease the deployment and run of your spiders in production.
 - A built-in Web service for monitoring and controlling your bot
 - A Telnet console for hooking into a Python console running inside your Scrapy process, to introspect and debug your crawler
 - Logging facility that you can hook on to for catching errors during the scraping process.
 - Support for crawling based on URLs discovered through Sitemaps [http://www.sitemaps.org]
 - A caching DNS resolver”

##scrapy导读

 1. 创建一个Scrapy项目
 2. 定义要提取的Item
 3. 写蜘蛛
 4. 写Item Pipline 保存数据

###创建Scrapy项目

    scrapy startproject tutorial

###定义要提取的Item
通过`scrapy.Item`类定义Item
通过`scrapy.Field`定义属性

##我们的第一个蜘蛛

蜘蛛定义了开始链接、接下去的链接以及数据提取规则

通过继承`scrapy.Spider`并且定义三个强制属性：

 - `name`用来标识蜘蛛，必须唯一。
 - `start_urls`蜘蛛开始爬取的的链接列表。
 - `parse()`用来解析返回值并提取数据和更多的跟踪链接。在每个开始链接返回内容之后被调用，参数是网页返回值。
 
原理：Scrapy给每个开始连接创建`scrapy.Request`对象，并且将parse方法赋给它们作为回调函数。
这些请求被调度，然后执行，并返回`scrapy.http.Response`对象，然后将该对象返回给parse()方法。

##提取数据
###介绍选择器
有好多方式可以从网页上提取数据。Scrapy用的是基于Xpath和CSS选择器的机制，叫做Scrapy Selector。
Scrapy提供了`Selector`类和方便的快捷调用，这样就不用你每次实例化了。

你可以把Selector看成是代表文档中节点的对象。所以，第一个被初始化的selector代表根节点即整个文档。

Selector有四个基本的方法

 - Xpath():返回selector的列表，每个代表根据xpath表达式选择到的节点
 - css():返回selector的列表
 - extract():返回一个Unicode字符串
 - re():返回一个Unicode字符串列表，根据参数正则表达式
 
在shell里测试选择器
使用Scrapy shell的前提是你安装了 IPython

要想启动shell，转到项目根目录，执行：

    scrapy shell 'http://www.dmoz.org/Computer/'
    
注意：要将url用引号引起来。

shell 加载完成之后，你将得到返回值，它被赋给一个本地response对象，如果你输入`respones.body`你会看到返回值的body，如果你输入`response.headers`你会看到返回头。

更重要的是，如果你输入`response.selector`你会得到一个selector对象。你可以用它来查询response。快捷引用`response.xpath()` 和 `response.css()` 分别对应 `response.selector.xpath()` 和 `response.selector.css()`”。

提取数据
例如：

    sel.xpath('//ul/li/text()').extract()
    
每个`.xpath()`调用都会返回一个selector的列表，我们可以对其中的每个节点再深入的调用`.xpath()`，如下：

    for sel in response.xpath('//ul/li')
        title=sel.xpath('a/text()').extract()
        
使用Item
Item 对象是自定义的Python dict。scrapy期望把抓到的数据以Item对象的形式返回。

