---
layout: post
category : lessons
title: "casperjs 调试"
tagline: "Supporting tagline"
tags : [casperjs]
---


一些关于调试casper脚本的小技巧：

- 使用`verbose`模式
- 事件钩子
- 输出序列化值到控制台
- 给你的闭包命名

##使用`verbose`模式

默认情况下，Casper实例不会输出任何东西到控制台。这使得在新建脚本或者调试脚本过程中非常令人困惑，所以最好是用下面代码作为开头：

    var casper = require('casper').create({
        verbose: true,
        logLevel: "debug"
    });


`verbose`选项会告诉Casper，把所有在`logLevel`以上的日志消息输出到标准输出，这样你就可以跟踪每一步了。

   
警告：这样的输出结果中可能会带有敏感信息，在生产环境中请谨慎使用。


##事件钩子

事件是CasperJS非常强大的特性。

一些事件可以用来调试你的脚本：

- `http.status.XXX`事件，当每次有资源返回`HTTP code`为`XXX`时就会被触发
- `remote.alert`事件，每次客户端调用`alert()`时触发
- `remote.message`事件，每次有消息送到客户端控制台时触发
- `step.added`事件，每次有新步骤加入到堆栈时触发
- 等等


监听事件非常简单：

    casper.on('http.status.404', function(resource) {
        this.log('Hey, this one is 404: ' + resource.url, 'warning');
    });


查看完整[事件列表](http://docs.casperjs.org/en/latest/events-filters.html#events-list)


##输出序列化值到控制台

有时候，查看一个变量，尤其是对象是非常有用的。`utils_dump()`函数可以做到：

    require('utils').dump({
        foo: {
            bar: 42
        },
    });

注意：`utils_dump()`不能序列化函数和复杂的循环结构。


##给你的闭包命名

最简单但是非常有效的实践方式就是：总是给闭包命名：

**难跟踪:**


    casper.start('http://foo.bar/', function() {
        this.evaluate(function() {
            // ...
        });
    });


**容易跟踪:**


    casper.start('http://foo.bar/', function afterStart() {
        this.evaluate(function evaluateStuffAfterStart() {
            // ...
        });
    });


用上面的方式，每次失败时，闭包名字都会在调用堆栈中打印出来，这样你就能轻松地定位问题

   
注意：当然，这招在其他javascript项目中同样适用。