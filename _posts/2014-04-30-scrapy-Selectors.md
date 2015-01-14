---
layout: post
category : scrapy
title: "scrapy Selectors"
tagline: "Supporting tagline"
tags : [scrapy]
---
原文地址：[http://doc.scrapy.org/en/latest/topics/selectors.html](http://doc.scrapy.org/en/latest/topics/selectors.html)

在爬取网页的过程中最常见的任务就是从HTML中提取数据，有很多库可以做到这一点：

 - BeautifulSoup 缺点：慢
 - lxml

scrapy 有自己的机制来提取数据，叫做selectors 因为它们通过xpath或css 选择文档的一部分。

scrapy selectors 构建在lxml之上，也就是说二者在速度和准确性上类似。但是scrapy selectors 的API比lxml的API简单很多，因为lxml除了可用来选择标记文档，还可以干其他很多工作。
 
##使用selectors
###构建selectors
Scrapy selectors 是Selector类的实例。通过传入`text`或`TextResponse` 对象构建。它会根据输入类型自动选择最优解析规则（XML还是HTML）
 

    >>> from scrapy.selector import Selector
    >>> from scrapy.http import HtmlResponse
    

由text构建

    >>> body = '<html><body><span>good</span></body></html>'
    >>> Selector(text=body).xpath('//span/text()').extract()
    [u'good']
    

由response构建

    >>> response = HtmlResponse(url='http://example.com', body=body)
    >>> Selector(response=response).xpath('//span/text()').extract()
    [u'good']
    

简便起见，response对象提供了一个`.selector`属性，方便引用

    >>> response.selector.xpath('//span/text()').extract()
    [u'good']
    

###使用selectors
为了展示如何使用selectors,我们使用scrapy shell (可以利用它的交互来测试)，和一个示例网页，HTML代码如下：

    <html>
     <head>
      <base href='http://example.com/' />
      <title>Example website</title>
     </head>
     <body>
      <div id='images'>
       <a href='image1.html'>Name: My image 1 <br /><img src='image1_thumb.jpg' /></a>
       <a href='image2.html'>Name: My image 2 <br /><img src='image2_thumb.jpg' /></a>
       <a href='image3.html'>Name: My image 3 <br /><img src='image3_thumb.jpg' /></a>
       <a href='image4.html'>Name: My image 4 <br /><img src='image4_thumb.jpg' /></a>
       <a href='image5.html'>Name: My image 5 <br /><img src='image5_thumb.jpg' /></a>
      </div>
     </body>
    </html>
    

首先，打开shell:

    scrapy shell http://doc.scrapy.org/en/latest/_static/selectors-sample1.html
    
shell 加载完之后，你就可以使用response变量了，它有属性`selector`。

由于我们解析的是HTML，所以selector会自动使用一个HTML解析器。

我们先通过xpath选择title标签里的文本

    >>> response.selector.xpath('//title/text()')
    [<Selector (text) xpath=//title/text()>]
    

由于通过xpath和css选择太常见了，所以response包含了两个快捷引用：`response.xpath()`和`response.css()`:

    >>> response.xpath('//title/text()')
    [<Selector (text) xpath=//title/text()>]
    >>> response.css('title::text')
    [<Selector (text) xpath=//title/text()>]
    

`xpath()`和`css()`方法会返回一个`SelectorList`实例，它是一系列新selectors 的列表。这样设计方便你快速选取嵌套的数据。

    >>> response.css('img').xpath('@src').extract()
    [u'image1_thumb.jpg',
     u'image2_thumb.jpg',
     u'image3_thumb.jpg',
     u'image4_thumb.jpg',
     u'image5_thumb.jpg']
    

为了提取文本数据，你必须调用selector的`extract()`方法，如下：

    >>> response.xpath('//title/text()').extract()
    [u'Example website']
    

css selectors 可以通过css3伪元素来选择文本或属性节点：

    >>> response.css('title::text').extract()
    [u'Example website']
    

下面我们将提取base URL  和一些图片链接
    
    >>> response.xpath('//base/@href').extract()
    [u'http://example.com/']
    
    >>> response.css('base::attr(href)').extract()
    [u'http://example.com/']
    
    >>> response.xpath('//a[contains(@href, "image")]/@href').extract()
    [u'image1.html',
     u'image2.html',
     u'image3.html',
     u'image4.html',
     u'image5.html']
    
    >>> response.css('a[href*=image]::attr(href)').extract()
    [u'image1.html',
     u'image2.html',
     u'image3.html',
     u'image4.html',
     u'image5.html']
    
    >>> response.xpath('//a[contains(@href, "image")]/img/@src').extract()
    [u'image1_thumb.jpg',
     u'image2_thumb.jpg',
     u'image3_thumb.jpg',
     u'image4_thumb.jpg',
     u'image5_thumb.jpg']
    
    >>> response.css('a[href*=image] img::attr(src)').extract()
    [u'image1_thumb.jpg',
     u'image2_thumb.jpg',
     u'image3_thumb.jpg',
     u'image4_thumb.jpg',
     u'image5_thumb.jpg']
    

###嵌套selectors
`xpath()`和`css()`返回一个selectors的列表，所以你可以调用这些selector的选取方法，如下：
    
    >>> links = response.xpath('//a[contains(@href, "image")]')
    >>> links.extract()
    [u'<a href="image1.html">Name: My image 1 <br><img src="image1_thumb.jpg"></a>',
     u'<a href="image2.html">Name: My image 2 <br><img src="image2_thumb.jpg"></a>',
     u'<a href="image3.html">Name: My image 3 <br><img src="image3_thumb.jpg"></a>',
     u'<a href="image4.html">Name: My image 4 <br><img src="image4_thumb.jpg"></a>',
     u'<a href="image5.html">Name: My image 5 <br><img src="image5_thumb.jpg"></a>']
    
    >>> for index, link in enumerate(links):
    ... args = (index, link.xpath('@href').extract(), link.xpath('img/@src').extract())
    ... print 'Link number %d points to url %s and image %s' % args
    
    Link number 0 points to url [u'image1.html'] and image [u'image1_thumb.jpg']
    Link number 1 points to url [u'image2.html'] and image [u'image2_thumb.jpg']
    Link number 2 points to url [u'image3.html'] and image [u'image3_thumb.jpg']
    Link number 3 points to url [u'image4.html'] and image [u'image4_thumb.jpg']
    Link number 4 points to url [u'image5.html'] and image [u'image5_thumb.jpg']
    

###使用selector的正则
Selector有一个`.re()`方法用来在提取数据时使用正则。不像xpath()和css(),re()方法返回一个Unicode字符串，所以你不能构建嵌套的.re()调用。

    >>> response.xpath('//a[contains(@href, "image")]/text()').re(r'Name:\s*(.*)')
    [u'My image 1',
     u'My image 2',
     u'My image 3',
     u'My image 4',
     u'My image 5']
    

###使用相对xpath
需要记住的是，如果你使用嵌套选择器，用到了以/开头的xpath，那么该xpath will be absolute to the document and not relative to the Selector you’re calling it from

例如：你想提取<div>元素中所有的<p>元素。首先，你要得到所有的<div>：

	>>> divs = response.xpath('//div')

接下来，你可能会试图使用下面的方法，下面的方法是错误的，因为它实际上提取的是document下所有的<p>元素，而不只是那些在<div>里的元素：

    >>> for p in divs.xpath('//p'):  # this is wrong - gets all <p> from the whole document
    ... print p.extract()
    

正确的方式应该是：注意前缀 .//p

    >>> for p in divs.xpath('.//p'):  # extracts all <p> inside
    ... print p.extract()
    

另一种常见的方式是提取所有直属<p>子元素
    
    >>> for p in divs.xpath('p'):
    ... print p.extract()
    

###使用EXSLT 扩展

###一些有关Xpath的tip
##Built-in Selectors reference