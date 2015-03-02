---
layout: post
category : 
title: "CasperJS 命令行"
tagline: "Supporting tagline"
tags : [CasperJS]
---
casperjs内置一个命令行解析器，基于PhantomJS 命令行构建，放在`cli`模块下；它会把命令行传过来的参数转换成有位置属性的参数和命名的选项（it exposes passed arguments as **positional ones** and **named options**）

但你无需担心调用`cli`模块的解析API，casper实例包含一个可以直接使用的`cli`属性，通过它你可以轻松获取这些参数。

先来看下面这段简单地casper 脚本：

    var casper = require("casper").create();

    casper.echo("Casper CLI passed args:");
    require("utils").dump(casper.cli.args);

    casper.echo("Casper CLI passed options:");
    require("utils").dump(casper.cli.options);

    casper.exit();

运行结果：

    $ casperjs test.js arg1 arg2 arg3 --foo=bar --plop anotherarg
    Casper CLI passed args: [
        "arg1",
        "arg2",
        "arg3",
        "anotherarg"
    ]
    Casper CLI passed options: {
        "casper-path": "/Users/niko/Sites/casperjs",
        "cli": true,
        "foo": "bar",
        "plop": true
    }

注意：上面出现的两个选项`casper-path` 和 `cli` ，是通过`casperjs` Python executable传给casper脚本的。

获取，检查或是丢弃(drop)参数：

    var casper = require("casper").create();
    casper.echo(casper.cli.has(0));
    casper.echo(casper.cli.get(0));
    casper.echo(casper.cli.has(3));
    casper.echo(casper.cli.get(3));
    casper.echo(casper.cli.has("foo"));
    casper.echo(casper.cli.get("foo"));
    casper.cli.drop("foo");
    casper.echo(casper.cli.has("foo"));
    casper.echo(casper.cli.get("foo"));
    casper.exit();

运行结果:


    $ casperjs test.js arg1 arg2 arg3 --foo=bar --plop anotherarg
    true
    arg1
    true
    anotherarg
    true
    bar
    false
    undefined

提示：如果你想检查是否有参数或者选项传递给脚本，可以用下面方式：


       // 去掉Python executable传递的默认选项 
       casper.cli.drop("cli");
       casper.cli.drop("casper-path");

       if (casper.cli.args.length === 0 && Object.keys(casper.cli.options).length === 0) {
           casper.echo("No arg nor option passed").exit();
       }

#casperjs原生的选项
`casperjs`命令有三个可用选项：

- ``--direct``: 打印log信息到控制台
- ``--log-level=[debug|info|warning|error]`` 设定log level
- ``--engine=[phantomjs|slimerjs]`` 选择浏览器引擎. CasperJS
  支持基于Webkit的 PhantomJS (默认), 和基于Gecko的 SlimerJS.

警告：`--direct`从1.1版本标记为deprecated。`--direct` 选项更名为 `--verbose`, 尽管 `--direct` 还可以工作，但是已经被视为deprecated

例如：

    $ casperjs --verbose --log-level=debug myscript.js


你还可以使用所有PhantomJs标准命令行选项，就像在phantomjs 脚本上使用一样：

    $ casperjs --web-security=no --cookies-file=/tmp/mycookies.txt myscript.js

提示：通过`phantomjs --help`查看可以用的phantomjs命令行选项。SlimerJS支持几乎和PhantomJS 一样的选项。

##原始参数值

默认情况下，`cli`对象会处理传进来的每个参数，将它们转成最合适类型，例如：

    var casper = require('casper').create();
    var utils = require('utils');

    utils.dump(casper.cli.get('foo'));

    casper.exit();

如果用下面方式运行上面代码：

    $ casperjs c.js --foo=01234567
    1234567

如你所见，`01234567`值被转换成了一个  *Number*.

有时候，你就想得到原本的字符串，那么你可以使用`cli`对象的`raw`属性，它包含着原始值：

    var casper = require('casper').create();
    var utils = require('utils');

    utils.dump(casper.cli.get('foo'));
    utils.dump(casper.cli.raw.get('foo'));

    casper.exit();

例如：

    $ casperjs c.js --foo=01234567
    1234567
    "01234567"





