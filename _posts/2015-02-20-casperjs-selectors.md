---
layout: post
category : lessons
title: "casperjs 选择器"
tagline: "Supporting tagline"
tags : [casperjs]
---


CasperJS使用了大量的选择器，方便你操作DOM，以及透明的使用CSS3或者XPath表达式。

后面的例子都基于下面的HTML结构：

    <!doctype html>
    <html>
    <head>
        <meta charset="utf-8">
        <title>My page</title>
    </head>
    <body>
        <h1 class="page-title">Hello</h1>
        <ul>
            <li>one</li>
            <li>two</li>
            <li>three</li>
        </ul>
        <footer><p>©2012 myself</p></footer>
    </body>
    </html>



##CSS3

默认情况下，CasperJS接受CSS3选择器字符串来检索DOM元素。

要检索元素`<h1 class="page-title">`是否存在,你可以用下面的方式：

    var casper = require('casper').create();

    casper.start('http://domain.tld/page.html', function() {
        if (this.exists('h1.page-title')) {
            this.echo('the heading exists');
        }
    });

    casper.run();


或者使用测试框架：

    casper.test.begin('The heading exists', 1, function suite(test) {
        casper.start('http://domain.tld/page.html', function() {
            test.assertExists('h1.page-title');
        }).run(function() {
            test.done();
        });
    });


有一些方便的测试方法也依赖于选择器：

    casper.test.begin('Page content tests', 3, function suite(test) {
        casper.start('http://domain.tld/page.html', function() {
            test.assertExists('h1.page-title');
            test.assertSelectorHasText('h1.page-title', 'Hello');
            test.assertVisible('footer');
        }).run(function() {
            test.done();
        });
    });



##XPath

你还可以使用XPath表达式：

    casper.start('http://domain.tld/page.html', function() {
        this.test.assertExists({
            type: 'xpath',
            path: '//*[@class="plop"]'
        }, 'the element exists');
    });


为了方便使用和阅读XPath表达式，`casper`模块提供了一个辅助方法`selectXPath`:

    var x = require('casper').selectXPath;

    casper.start('http://domain.tld/page.html', function() {
        this.test.assertExists(x('//*[@id="plop"]'), 'the element exists');
    });

   
警告：在CasperJS里使用XPath唯一的限制就是当你想用`casper.fill()`方法填充file fields时；PhantomJS原生[upload方法](https://github.com/ariya/phantomjs/wiki/API-Reference#wiki-webpage-uploadFile)只允许使用CSS3选择器。
