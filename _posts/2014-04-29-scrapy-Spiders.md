---
layout: post
category : scrapy
title: "scrapy Spiders"
tagline: "Supporting tagline"
tags : [scrapy]
---
原文地址：[http://doc.scrapy.org/en/latest/topics/spiders.html](http://doc.scrapy.org/en/latest/topics/spiders.html)

蜘蛛爬行过程大体如下：

 1. 以给起始URLs生成requests开始，并给这些requests指定回调函数，当response下载完后，调用回调函数。
 第一个request是通过调用`start_requests()`方法取得。`start_requests()`默认情况下会给指定在`start_urls`里的URL生成request，并将`parse`方法作为request的回调函数。
 2. 在回调函数中，你解析response并返回item 对象或者request对象或者an iterable of both 。这些request对象同样也会包含回调函数（可以是一样的），接下来，scrapy会发起这些请求，并用指定的回调函数处理response。
 3. 在回调函数中，解析网页内容，通常使用Selectors(你也可使用BeautifulSoup, lxml 或其它你想用的),并用提取的数据生成item。
 4. 最后蜘蛛返回的这些items通常会被存到数据库（用一些item pipeline ）或用Feed export写到文件中。

##spider arguments 
蜘蛛可以接受参数用来改变它们的行为。参数常用来限制蜘蛛爬取某一板块的内容。但其实，参数可以用来配置蜘蛛所有的功能。

spider arguments 通过`crawl`命令的`-a`选项传递，例如：

    scrapy crawl myspider -a category=electronics
    
spider会在它们的构造函数里接收参数：


	import scrapy

	class MySpider(scrapy.Spider):
    	name = 'myspider'

    	def __init__(self, category=None, *args, **kwargs):
        	super(MySpider, self).__init__(*args, **kwargs)
        	self.start_urls = ['http://www.example.com/categories/%s' % category]
        	# ...


也可以通过Scrapyd schedule.json API 传递参数。参见Scrapyd。

##内建蜘蛛
scrapy提供了很多通用蜘蛛，你可以直接从它们继承。
下面的例子假定你有一个项目，且在myproject.items 模块下有TestItem定义：

	import scrapy

	class TestItem(scrapy.Item):
    	id = scrapy.Field()
    	name = scrapy.Field()
    	description = scrapy.Field()


###Spider


	class scrapy.spider.Spider


该类是所有蜘蛛的父类，不论是scrapy自带的还是你自己写的蜘蛛，最终都需要继承于此类。它并不提供任何特殊的功能，只是请求给定的start_url/start_requests，等response后调用回调函数解析response。

 - `name`蜘蛛的名字。scrapy用名字来定位蜘蛛，所以它必须唯一。但是对于一个蜘蛛类你可以实例化多个实例。名字通常为域名。
 - `allowed_domains`可选的域名列表。如果`OffsiteMiddleware`启用的话，那么不属于这个域名列表里的requests和URLs都不会被跟踪。
 - `start_urls`当没有指定特定的URL时，蜘蛛会从这个URLs列表开始爬取。接下来的链接会从start URLs对应的页面中生成。
 - `start_requests()`该方法必须返回一个iterable。只有当没有指定特定URLs时才会调用该方法。如果指定了特定URL，那么`make_requests_from_url(url)`会被调用。
 	
	该方法只会被scrapy调用一次，所以将它实现为一个生成器是安全的。默认的实现是：为每个`start_urls`里的链接调用`make_requests_from_url(url)`生成request。
	如果你想改变初始request，这个方法就是需要被重写的。例如：你需要先POST request 来登录，可以这么做：

	  	def start_requests(self):
		  	return [scrapy.FormRequest("http://www.example.com/login",
                               formdata={'user': 'john', 'pass': 'secret'},
                               callback=self.logged_in)]

	  	def logged_in(self, response):
		  	# here you would extract links to follow and return Requests for
		  	# each of them, with another callback
		  	pass

 - `make_requests_from_url(url)`该方法接受一个url,返回一个Request对象（或者一个request对象列表）用来爬取。在`start_requests()`中，它被用来构建初始request。它通常被用来将url转换为request。
 除非被重写，否则该方法返回request，并把`parse()`函数作为其回调函数。
 - `parse(response)`当没有指定别的回调函数时，该方法是默认的用来处理response的回调函数。
 该方法处理response，返回提取的数据和/或更多的URL。其它的Request的毁掉函数也必须如此。必须返回iterable request and/ or item objects.

	 Parameters:response (:class:~scrapy.http.Response`) – the response to parse


- `log(message[,level,component])`使用`scrapy.log.msg()`函数log。
- `closed(reason)`蜘蛛关闭时被调用。 This method provides a shortcut to signals.connect() for the `spider_closed` signal.

示例：

	import scrapy

	class MySpider(scrapy.Spider):
    	name = 'example.com'
    	allowed_domains = ['example.com']
    	start_urls = [
        	'http://www.example.com/1.html',
        	'http://www.example.com/2.html',
        	'http://www.example.com/3.html',
    	]

    	def parse(self, response):
        	self.log('A response from %s just arrived!' % response.url)
        

 
下面是,从一个回调函数中返回多个Requests 和Item的例子：

	import scrapy
	from myproject.items import MyItem

	class MySpider(scrapy.Spider):
    	name = 'example.com'
    	allowed_domains = ['example.com']
    	start_urls = [
        	'http://www.example.com/1.html',
        	'http://www.example.com/2.html',
        	'http://www.example.com/3.html',
    	]

    	def parse(self, response):
        	for h3 in response.xpath('//h3').extract():
            	yield MyItem(title=h3)

        	for url in response.xpath('//a/@href').extract():
            	yield scrapy.Request(url, callback=self.parse)

###CrawlSpider

	class scrapy.contrib.spiders.CrawlSpider

这是最常用的蜘蛛，因为它提供了一种方便的机制用来跟踪链接，只需定义一系列规则。除了从`Spider`继承的属性外，它还新增了一个属性：

 - `rules`一系列Rule对象。每个Rule对象定义了一种确定的行为。当同一个链接符合多条规则时，会使用第一条。

它还新增一个可重写的方法：

 - `parse_start_url(response)`这个方法会在start_url response 后调用。用来解析初始response，但它也必须返回item 对象或Request对象，或包含它们的iterable 

####Crawling rules

	class scrapy.contrib.spiders.Rule(link_extractor, callback=None, cb_kwargs=None, follow=None, process_links=None, process_request=None)


 - `link_extractor`是`Link Extractor`对象，定义了如何从每个爬取的页面中提取链接。
 - `callback`is a callable or a string（函数名）在指定的`link_extractor`提取连接后调用。该回调函数接受一个response作为第一个参数，它必须返回一个包含item and/or request 对象的列表。
 注意：当写蜘蛛规则时，不要使用parse作为回调函数，因为`CrawlSpider`本身用到parse来实现自身逻辑。所以一旦你重写了parse函数，蜘蛛就不能正常工作了。
 - `cb_kwargs`包含关键词参数的字典，会被传递给回调函数。
 - `follow`布尔值。用来指定是否跟踪当前规则提取到的链接。如果没有指定回调函数，那么默认为true,否则默认为false。
 - `process_links`is a callable, or a string。用来过滤提取到的链接
 - `process_request`is a callable, or a string。用来过滤request

示例：

	import scrapy
	from scrapy.contrib.spiders import CrawlSpider, Rule
	from scrapy.contrib.linkextractors import LinkExtractor

	class MySpider(CrawlSpider):
    	name = 'example.com'
    	allowed_domains = ['example.com']
    	start_urls = ['http://www.example.com']

    	rules = (
        	# Extract links matching 'category.php' (but not matching 'subsection.php')
        	# and follow links from them (since no callback means follow=True by default).
        	Rule(LinkExtractor(allow=('category\.php', ), deny=('subsection\.php', ))),

        	# Extract links matching 'item.php' and parse them with the spider's method parse_item
        	Rule(LinkExtractor(allow=('item\.php', )), callback='parse_item'),
    )

    	def parse_item(self, response):
        	self.log('Hi, this is an item page! %s' % response.url)
        	item = scrapy.Item()
        	item['id'] = response.xpath('//td[@id="item_id"]/text()').re(r'ID: (\d+)')
        	item['name'] = response.xpath('//td[@id="item_name"]/text()').extract()
        	item['description'] = response.xpath('//td[@id="item_description"]/text()').extract()
        	return item

 
###XMLFeedSpider
###CSVFeedSpider
###SitemapSpider
 