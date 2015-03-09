---
layout: post
category : lessons
title: "casperjs 日志"
tagline: "Supporting tagline"
tags : [casperjs]
---

CasperJS 可以通过`casper.log()`记录日志，日志有四个级别：

- ``debug``
- ``info``
- ``warning``
- ``error``


示例：

    var casper = require('casper').create();
    casper.log('plop', 'debug');
    casper.log('plip', 'warning');


现在，需要区分两个概念，记录日志和显示日志，默认情况下，CasperJS不会把日志打印到标准输出。要想打印出来，你必须启用`verbose`选项：

    var casper = require('casper').create({
        verbose: true
    });


同时，casper默认配置过滤`error`级别以下的日志，你可以通过`logLevel`选项重写这一配置：

    var casper = require('casper').create({
        verbose: true,
        logLevel: 'debug'
    });


还可以通过`Casper.logs`属性将日志转存成JSON格式：

    var casper = require('casper').create({
    // ...
    casper.run(function() {
        require('utils').dump(this.logs);
        this.exit();
    });


最后，如果你使用`verbose`选项，将日志输出到标准输出，你会得到彩色输出：

    var casper = require('casper').create({
        verbose: true,
        logLevel: 'debug'
    })
    casper.log('this is a debug message', 'debug');
    casper.log('and an informative one', 'info');
    casper.log('and a warning', 'warning');
    casper.log('and an error', 'error');
    casper.exit();


结果大致如下：

![http://docs.casperjs.org/en/latest/_images/logoutput.png](http://docs.casperjs.org/en/latest/_images/logoutput.png)

   
提示：CasperJS不会在文件系统记录日志，如果需要的话，你得自己实现。