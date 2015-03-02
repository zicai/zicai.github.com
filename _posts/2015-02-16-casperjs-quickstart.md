---
layout: post
category : 
title: "CasperJS 快速开始"
tagline: "Supporting tagline"
tags : [CasperJS]
---
CasperJS安装完成后，就可以开始写脚本了。你可以使用javascript或coffeescript


##一个简单实例

新建一个`sample.js`文件，内容如下：

    var casper = require('casper').create();

    casper.start('http://casperjs.org/', function() {
        this.echo(this.getTitle());
    });

    casper.thenOpen('http://phantomjs.org', function() {
        this.echo(this.getTitle());
    });

    casper.run();

运行:

    $ casperjs sample.js

输出结果大体如下：

    $ casperjs sample.js
    CasperJS, a navigation scripting and testing utility for PhantomJS
    PhantomJS: Headless WebKit with JavaScript API

上面的程序做了几件事：

   1. 创建了一个Casper实例
   2. 开启casper，并打开`http://casperjs.org/`
   3. 等到页面加载完成，输出页面标题
   4. 然后打开另一个URL, `http://phantomjs.org/`
   5. 新页面加载完成后，同样是输出页面标题
   6. 执行整个过程


##爬取谷歌

接下来的实例，我们用谷歌连续搜索两个单词casperjs和phantomjs，然后将搜索结果的url聚合到一个数组，最后输出到控制台。

新建文件`googlelinks.js`，内容如下：


    var links = [];
    var casper = require('casper').create();

    function getLinks() {
        var links = document.querySelectorAll('h3.r a');
        return Array.prototype.map.call(links, function(e) {
            return e.getAttribute('href');
        });
    }

    casper.start('http://google.fr/', function() {
        // search for 'casperjs' from google form
        this.fill('form[action="/search"]', { q: 'casperjs' }, true);
    });

    casper.then(function() {
        // aggregate results for the 'casperjs' search
        links = this.evaluate(getLinks);
        // now search for 'phantomjs' by filling the form again
        this.fill('form[action="/search"]', { q: 'phantomjs' }, true);
    });

    casper.then(function() {
        // aggregate results for the 'phantomjs' search
        links = links.concat(this.evaluate(getLinks));
    });

    casper.run(function() {
        // echo results in some pretty fashion
        this.echo(links.length + ' links found:');
        this.echo(' - ' + links.join('\n - ')).exit();
    });

运行：


    $ casperjs googlelinks.js
    20 links found:
     - https://github.com/n1k0/casperjs
     - https://github.com/n1k0/casperjs/issues/2
     - https://github.com/n1k0/casperjs/tree/master/samples
     - https://github.com/n1k0/casperjs/commits/master/
     - http://www.facebook.com/people/Casper-Js/100000337260665
     - http://www.facebook.com/public/Casper-Js
     - http://hashtags.org/tag/CasperJS/
     - http://www.zerotohundred.com/newforums/members/casper-js.html
     - http://www.yellowpages.com/casper-wy/j-s-enterprises
     - http://local.trib.com/casper+wy/j+s+chinese+restaurant.zq.html
     - http://www.phantomjs.org/
     - http://code.google.com/p/phantomjs/
     - http://code.google.com/p/phantomjs/wiki/QuickStart
     - http://svay.com/blog/index/post/2011/08/31/Paris-JS-10-%3A-Introduction-%C3%A0-PhantomJS
     - https://github.com/ariya/phantomjs
     - http://dailyjs.com/2011/01/28/phantoms/
     - http://css.dzone.com/articles/phantom-js-alternative
     - http://pilvee.com/blog/tag/phantom-js/
     - http://ariya.blogspot.com/2011/01/phantomjs-minimalistic-headless-webkit.html
     - http://www.readwriteweb.com/hack/2011/03/phantomjs-the-power-of-webkit.php




###CoffeeScript 版本


你也可以用coffeescript来写casper 脚本


    getLinks = ->
      links = document.querySelectorAll "h3.r a"
      Array::map.call links, (e) -> e.getAttribute "href"

    links = []
    casper = require('casper').create()

    casper.start "http://google.fr/", ->
      # search for 'casperjs' from google form
      @fill "form[action='/search']", q: "casperjs", true

    casper.then ->
      # aggregate results for the 'casperjs' search
      links = @evaluate getLinks
      # search for 'phantomjs' from google form
      @fill "form[action='/search']", q: "phantomjs", true

    casper.then ->
      # concat results for the 'phantomjs' search
      links = links.concat @evaluate(getLinks)

    casper.run ->
      # display results
      @echo links.length + " links found:"
      @echo(" - " + links.join("\n - ")).exit()


注意：文件要以`.coffee`作为扩展名


##一个测试实例

CasperJS同时也是一个测试框架，测试脚本和爬取脚本稍有区别，尽管它们共用大多数API。

一段简单地测试脚本:

    // hello-test.js
    casper.test.begin("Hello, Test!", 1, function(test) {
      test.assert(true);
      test.done();
    });

用`casperjs test` 子命令来运行上面的脚本。


    $ casperjs test hello-test.js
    Test file: hello-test.js
    # Hello, Test!
    PASS Subject is strictly true
    PASS 1 test executed in 0.023s, 1 passed, 0 failed, 0 dubious, 0 skipped.

注意：如你所见，在测试脚本中不需要创建casper实例，因为已经预先配置好了一个实例供你调用。




