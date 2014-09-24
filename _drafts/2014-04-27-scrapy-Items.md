---
layout: post
category : scrapy
title: scrapy Items
tagline: "Supporting tagline"
tags : [scrapy]
---
原文地址：[http://doc.scrapy.org/en/latest/topics/items.html](http://doc.scrapy.org/en/latest/topics/items.html)

爬取网页最主要的目的就是从非结构化的来源中，通常是网页，提取结构化的数据。为此，scrapy提供了item类。

Item是用来收集抓取数据的简单容器，它提供了dictionary-like api 可以方便的声明fields。

##声明item


	import scrapy

	class Product(scrapy.Item):
    	name = scrapy.Field()
    	price = scrapy.Field()
    	stock = scrapy.Field()
    	last_updated = scrapy.Field(serializer=str)

scrapy item 类似于Django modle ,只是scrapy item 更简单，因为它所有field 类型都一样。


##Item Field 
Field对象用来给每个域指定元数据。例如：上面例子展示的`last_updateed`域指定了`serializer`函数。

你可以指定任何的元数据，Field 对象接受的值并没有限制。所以，没有一个可用的元数据键的列表。每个键会被不同的组件使用到，并且只有这些组件知道。Field对象最主要的目的是提供一个方式将所有元数据定义到一个地方。


##使用item
下面是一些使用item的场景，用到了上面定义的Produc item。
###创建


	>>> product = Product(name='Desktop PC', price=1000)
	>>> print product
	Product(name='Desktop PC', price=1000)


###取值

	>>> product['name']
	Desktop PC
	>>> product.get('name')
	Desktop PC

	>>> product['price']
	1000

	>>> product['last_updated']
	Traceback (most recent call last):
    	...
	KeyError: 'last_updated'

	>>> product.get('last_updated', 'not set')
	not set

	>>> product['lala'] # getting unknown field
	Traceback (most recent call last):
    	...
	KeyError: 'lala'

	>>> product.get('lala', 'unknown field')
	'unknown field'

	>>> 'name' in product  # is name field populated?
	True

	>>> 'last_updated' in product  # is last_updated populated?
	False

	>>> 'last_updated' in product.fields  # is last_updated a declared field?
	True

	>>> 'lala' in product.fields  # is lala a declared field?
	False


###赋值

    >>> product['last_updated'] = 'today'
    >>> product['last_updated']
    today
    
    >>> product['lala'] = 'test' # setting unknown field
    Traceback (most recent call last):
    ...
    KeyError: 'Product does not support field: lala'


###取得所有填充的值

    >>> product.keys()
    ['price', 'name']
    
    >>> product.items()
    [('price', 1000), ('name', 'Desktop PC')]


###其它常见任务

拷贝item

    >>> product2 = Product(product)
    >>> print product2
    Product(name='Desktop PC', price=1000)
    
    >>> product3 = product2.copy()
    >>> print product3
    Product(name='Desktop PC', price=1000)


用item创建字典

    >>> dict(product) # create a dict from all populated values
    {'price': 1000, 'name': 'Desktop PC'}


用字典创建item

    >>> Product({'name': 'Laptop PC', 'price': 1500})
    Product(price=1500, name='Laptop PC')
    
    >>> Product({'name': 'Laptop PC', 'lala': 1500}) # warning: unknown field in dict
    Traceback (most recent call last):
    ...
    KeyError: 'Product does not support field: lala'


##扩展item
通过声明继承你原始的item，可以扩展item（给它添加更多的field，或者是改变一些field的元数据）。例如：

    class DiscountedProduct(Product):
    	discount_percent = scrapy.Field(serializer=str)
    	discount_expiration_date = scrapy.Field()


通过使用之前的field元数据再加上新的值，还可以扩展元数据或者是改变已经存在的值，例如：

	class SpecificProduct(Product):
    	name = scrapy.Field(Product.fields['name'],serializer=my_serializer)

添加或者替换serializer 元数据会保留之前已经存在的元数据值。

##item对象


	class scrapy.item.Item([arg])

返回一个新item,可以通过参数进行初始化（可选）。

item复制了标准dict api ,包括它的constructor。只是增加了一个fields属性。


	fields

一个包含了所有声明的field的dict，不只是已填充的。键是field name，值是Field对象。

##Field对象

	class scrapy.item.Field([arg])

Field类就是内置的dict类的别名，它并不提供额外的方法和属性。也就是说，Field对象就是纯python dict。


