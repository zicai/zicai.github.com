---
layout: post
category : scrapy
title: "scrapy Link Extractors"
tagline: "Supporting tagline"
tags : [scrapy]
---
原文地址：[http://doc.scrapy.org/en/latest/topics/link-extractors.html](http://doc.scrapy.org/en/latest/topics/link-extractors.html)

LinkExtractors 对象唯一的目的就是从网页上提取最终会被追踪的链接。

LinkExtractor 只有一个公开的方法 `extract_links`，它检索`response`对象，返回一个`scrapy.link.Link`对象列表。LinkExtractors 只会被实例化一次，但`extract_links`方法会被多次调用，以不同的response。

##内建的link extractors
所有link extractors类都在`scrapy.contrib.linkextractors`模块里
    
    from scrapy.contrib.linkextractors import LinkExtractor
    
###LxmlLinkExtractor
```python
class scrapy.contrib.linkextractors.lxmlhtml.LxmlLinkExtractor(allow=(), deny=(), allow_domains=(), deny_domains=(), deny_extensions=None, restrict_xpaths=(), tags=('a', 'area'), attrs=('href', ), canonicalize=True, unique=True, process_value=None)
```

推荐使用LxmlLinkExtractor，因为它带有易用的筛选选项。它基于lxml的 HTMLParser实现。

参数：

 - `allow`一个正则表达式（或一组正则表达式），只有匹配的才能被
 - `deny`
 - `allow_domains`
 - `deny_domains`
 - `deny_extensions`
 - `restrict_xpaths`
 - `tags`
 - `attrs`
 - `canonicalize`
 - `unique`
 - `process_value`