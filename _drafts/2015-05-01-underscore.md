---
layout: post
category : lessons
title: "underscore学习"
tagline: "Supporting tagline"
tags : [underscore]
---
##集合

Note: Collection functions work on arrays, objects, and array-like objects such as arguments, NodeList and similar. But it works by duck-typing, so avoid passing objects with a numeric length property. It's also good to note that an each loop cannot be broken out of — to break, use _.find instead.

###遍历
each
map
pluck
invoke
###查找
find

where
findWhere

###过滤
filter
reject
partition
###判定
every
some
contains

###随机
shuffle
sample

###其它

reduce
reduceRight
max
min
size

sortBy
groupBy
indexBy
countBy
toArray



##数组

###去掉数组中部分元素
without
difference
compact
uniq
###返回指定位置元素
first
initial
last
rest
###多数组操作
union
intersection
difference
zip
unzip
###数组转对象
object

###查找索引
indexOf
lastIndexOf
sortedIndex
findIndex
findLastIndex

###创建数组
range
###其它
flatten


###数组变对象


##对象
###获取属性
keys
allKeys
findkey
has
###获取属性值
values
property
propertyOf
###转换属性值
mapObject
###键值对换
invert
###对象变数组
pairs
###获取对象所有方法
functions

###扩展对象
extend
extendOwn
###精简对象
pick
omit

###判断
matcher
isEqual
isMatch
isEmpty
isFinite

###判断数据类型
isElement
isArray
isObject
isArguments
isFunction
isString
isNumber
isBoolean
isDate
isRegExp
isError
isNaN
isNull
isUndefined

###其它
defaults
clone
tap


























