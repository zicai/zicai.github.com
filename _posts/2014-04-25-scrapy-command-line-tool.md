---
layout: post
category : scrapy
title: scrapy 命令行工具
tagline: "Supporting tagline"
tags : [scrapy]
---
原文地址：[http://doc.scrapy.org/en/latest/topics/commands.html](http://doc.scrapy.org/en/latest/topics/commands.html)

scrapy 通过command-line tool来控制，为了和subcommands区分，我们把前者叫做“scrapy tool”,把后者叫做“commands”或者“scrapy commands”。

##scrapy默认项目结构
scrapy.cfg文件所在目录是项目的根目录。该文件包含了Python模块的名字，它用来定义项目的设置。

##使用scrapy tool
你可以不带参数的执行scrapy，它会打印一些哟用的帮助信息和可用的commands。

    Scrapy X.Y - no active project
    
    Usage:
      scrapy <command> [options] [args]
    
    Available commands:
      crawl         Run a spider
      fetch         Fetch a URL using the Scrapy downloader
    [...]
    
第一行会显示使用的scrapy的版本号和当前项目，前提是你在一个scrapy项目中。上面的例子，是在项目外执行scrapy。如果你在项目中执行，会得到类似如下结果：

    Scrapy X.Y - project: myproject
    
    Usage:
      scrapy <command> [options] [args]
    
    [...]

###创建项目
通常用scrapy tool干得第一件事就是创建项目：

    scrapy startproject myproject

该命令会在myproject目录下创建一个scrapy项目。

###控制项目
例如，创建一个新蜘蛛：

    scrapy genspider mydomain mydomain.com

有一些命令必须在项目中运行，另外的则不需要。还有一些命令在项目中运行和在项目外运行会稍有区别。

##可用的tool commands
查看所有的可用的命令：

    scrapy -h
    
查看单条命令的帮助：

    scrapy <command> -h
    
命令分为两类，一类是全局命令（不需要依赖某个项目），一类是项目命令（Project-only）

全局命令

 - startproject
 - settings
 - runspider
 - shell
 - fetch
 - view
 - version
 
项目命令

 - crawl
 - check
 - list
 - edit
 - parse
 - genspider
 - deploy
 - bench
 
###startproject
 - 语法：scrapy startproject <project_name>
 - 全局命令
 
它会在project_name目录下创建一个名为project_name的scrapy新项目。

###genspider

 - 语法：scrapy genspider [-t template] <name> <domain>
 - 项目命令
 
该命令可以方便快捷的基于预定义的模板创建蜘蛛，当然你也可以不用此命令，而是直接写代码创建。

    $ scrapy genspider -l  //列出可用模板
    Available templates:
      basic
      crawl
      csvfeed
      xmlfeed
    
    $ scrapy genspider -d basic  //显示某个具体模板
    import scrapy
    
    class $classname(scrapy.Spider):
        name = "$name"
        allowed_domains = ["$domain"]
        start_urls = (
            'http://www.$domain/',
            )
    
        def parse(self, response):
            pass
    
    $ scrapy genspider -t basic example example.com
    Created spider 'example' using template 'basic' in module:
      mybot.spiders.example


###crawl

 - 语法：scrapy crawl <spider>
 - 项目命令

用某个蜘蛛开始爬
 
###check

 - 语法：scrapy check [-l] <spider>
 - 项目命令

Run contract checks.

例如：


	$ scrapy check -l
	first_spider
  	* parse
  	* parse_item
	second_spider
  	* parse
  	* parse_item

	$ scrapy check
	[FAILED] first_spider:parse_item
	>>> 'RetailPricex' field is missing

	[FAILED] first_spider:parse
	>>> Returned 92 requests, expected 0..4


 ps:已经发生改变，以实际情况为准。
 
###list

 - 语法：scrapy list
 - 项目命令

列出当前项目中所有的蜘蛛。
 
###edit

 - 语法：scrapy edit <spider>
 - 项目命令

用EDITOR设置的编辑器编辑给定的蜘蛛。
 
###fetch

 - 语法：scrapy fetch <url>
 - 全局命令

注意url必须带http://
 
使用Scrapy downloader 下载指定url,并将内容输出到标准输出。

有意思的地方是：它会以蜘蛛真正下载网页的方式获取网页。例如：如果蜘蛛有USER_AGENT属性，那会重写默认的User Agent，它会使用USER_AGENT。

所以，该命令可以用来查看蜘蛛如何下载一个指定网页

如果你在项目外执行该命令，那么没有特定的蜘蛛行为会被应用到它上，它会使用默认的Scrapy downloader设置。


	$ scrapy fetch --nolog http://www.example.com/some/page.html
	[ ... html content here ... ]

	$ scrapy fetch --nolog --headers http://www.example.com/
	{'Accept-Ranges': ['bytes'],
	 'Age': ['1263   '],
 	'Connection': ['close     '],
 	'Content-Length': ['596'],
 	'Content-Type': ['text/html; charset=UTF-8'],
 	'Date': ['Wed, 18 Aug 2010 23:59:46 GMT'],
	'Etag': ['"573c1-254-48c9c87349680"'],
 	'Last-Modified': ['Fri, 30 Jul 2010 15:30:18 GMT'],
 	'Server': ['Apache/2.2.3 (CentOS)']}



 
###view

 - 语法：scrapy view <url>
 - 全局命令

scrapy会把蜘蛛看到的网页保存下来，并在浏览器中打开。因为有时候蜘蛛看到的网页和用户看到的网页不一样，所以它可以用来检查蜘蛛看到的并确认是你想要的。
 
###shell

 - 语法：scrapy shell [url]
 - 全局命令

以给定的url(如果指定了)启动Scrapy shell 。
 
###parse

 - 语法：scrapy parse <url> [options]
 - 项目命令

获取给定网页，并交给蜘蛛解析，如果给定了--callback，则用给定函数解析，否则用parse解析。

支持选项：

 - --spider=SPIDER: 跳过蜘蛛自动检测，强制使用指定蜘蛛
 - --a NAME=VALUE: 设置蜘蛛参数 (may be repeated)
 - --callback or -c: 用来解析response的回调函数
 - --pipelines: process items through pipelines
 - --rules or -r: use CrawlSpider rules to discover the callback (i.e. spider method) to use for parsing the response
 - --noitems: don’t show scraped items
 - --nolinks: don’t show extracted links
 - --nocolour: avoid using pygments to colorize the output
 - --depth or -d: depth level for which the requests should be followed recursively (default: 1)
 - --verbose or -v: display information for each depth level

 例如：
 

	$ scrapy parse http://www.example.com/ -c parse_item
	[ ... scrapy log lines crawling example.com spider ... ]

	>>> STATUS DEPTH LEVEL 1 <<<
	# Scraped Items  ------------------------------------------------------------
	[{'name': u'Example item',
	 'category': u'Furniture',
	 'length': u'12 cm'}]

	# Requests  -----------------------------------------------------------------
	[]


###settings

 - 语法：scrapy settings [options]
 - 全局命令

获取Scrapy 配置信息。

如果你在一个Scrapy 项目中，它会显示项目的配置。如果没在项目中，它会显示Scrapy 默认设置。
 
###runspider

 - 语法：scrapy runspider <spider_file.py>
 - 全局命令

运行一个包含蜘蛛的Python文件，而不用创建项目。
 
###version

 - 语法：scrapy version [-v]
 - 全局命令

显示scrapy 版本。如果带上 -v，则会显示Python ，Twisted以及平台信息，用于反馈bug。
 
###deploy

 - 语法：scrapy deploy [ <target:project> | -l <target> | -L ]
 - 项目命令

部署项目到一个Scrapyd server
 
###bench

 - 语法：scrapy bench
 - 全局命令

运行一个快速的性能测试。
 
##自定义项目命令