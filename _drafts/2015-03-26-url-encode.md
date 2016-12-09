---
layout: post
category : lessons
title: "URL 编码"
tagline: ""
tags : [encode]
---

## 为什么要有 URL 编码

URL 设计者们在设计 URL 时要解决的几个问题：

- 可移植性（portable）：URL 要统一的命名因特网上所有资源，也就意味可以通过不同协议（HTTP、SMTP、FTP 等）安全的传输这些资源。 安全传输意味着 URL 的传输不能丢失信息。由于不同的协议采用不同的传输机制，所以 URL 只能使用一些相对较小的、通用的安全字母表中的字符。
- 完整性：设计者们认识到，有时人们希望 URL 中包含除通用的安全字母表之外的二进制数据或者字符。因此，需要一种转义机制，先将不安全的字符编码为安全字符，再进行传输
- 可阅读性：除了希望 URL 可以被不同协议使用，还希望它能够供人类阅读。所以，即使那些不可见、不可打印字符能被程序所理解，也不能在 URL 中使用它们。

考虑到这些，就需要规定一个 URL 字符集和一种编码机制

## URL 字符集

URL 规范 [RFC 1738](http://www.ietf.org/rfc/rfc1738.txt) 将URL 中允许出现的字符限定为 ASCII 字符集的一个子集：

> "...Only alphanumerics [0-9a-zA-Z], the special characters "$-_.+!*'()," [not including the quotes - ed], and reserved characters used for their reserved purposes may be used unencoded within a URL."

> "只有字母和数字[0-9a-zA-Z]、一些特殊符号"$-_.+!*'(),"[不包括双引号]、以及某些保留字，才可以不经过编码直接用于URL。"

## 哪些字符需要编码

- URI中除了允许的字符之外的字符都必须用百分号编码
- 当保留字符用于其它目的时

## URL 编码机制
https://url.spec.whatwg.org/#percent-encoded-bytes

## JavaScript 与 URL 编码

- encodeURI() 和 decodeURI()
- encodeURIComponent() 和 decodeURIComponent


参考链接

- 《HTTP 权威指南》P38
- https://url.spec.whatwg.org/
- https://zh.wikipedia.org/wiki/%E7%99%BE%E5%88%86%E5%8F%B7%E7%BC%96%E7%A0%81
- http://www.blooberry.com/indexdot/html/topics/urlencoding.htm
