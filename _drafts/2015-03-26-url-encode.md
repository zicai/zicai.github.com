---
layout: post
category : lessons
title: "URL 编码"
tagline: ""
tags : [encode]
---

URL 规范 [RFC 1738](http://www.ietf.org/rfc/rfc1738.txt) 将URL 中允许出现的字符限定为 ASCII 字符集的一个子集：

> "...Only alphanumerics [0-9a-zA-Z], the special characters "$-_.+!*'()," [not including the quotes - ed], and reserved characters used for their reserved purposes may be used unencoded within a URL."

> "只有字母和数字[0-9a-zA-Z]、一些特殊符号"$-_.+!*'(),"[不包括双引号]、以及某些保留字，才可以不经过编码直接用于URL。"

而 HTML 则允许在文档中使用所有 ISO-8859-1（ISO-Latin）字符。到了 HTML 4，字符集扩展为 Unicode 字符集。

## 哪些字符需要编码

- URI中除了允许的字符之外的字符都必须用百分号编码
- 当保留字符用于其它目的时

## 如何编码
https://url.spec.whatwg.org/#percent-encoded-bytes

## JavaScript 与 URL 编码

- encodeURI() 和 decodeURI()
- encodeURIComponent() 和 decodeURIComponent


参考链接

- https://url.spec.whatwg.org/
- https://zh.wikipedia.org/wiki/%E7%99%BE%E5%88%86%E5%8F%B7%E7%BC%96%E7%A0%81
- http://www.blooberry.com/indexdot/html/topics/urlencoding.htm
