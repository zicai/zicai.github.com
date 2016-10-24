---
layout: post
category : lessons
title: "CasperJS 测试框架"
tagline: "Supporting tagline"
tags : [casperjs]
---


CasperJS包含自己的测试框架，提供了一系列能帮你简化webAPP测试的工具

警告：
	由于API设计，测试框架只能在	`casperjs test`子命令中使用：
	
- 如果你尝试在测试环境之外使用`casperjs.test`属性，会报错；
- 在1.1-bata3版本，在测试环境中，你不能重写预定义的`casper`实例



##单元测试


假设我们要给一个`Cow`对象写单元测试：

    function Cow() {
        this.mowed = false;
        this.moo = function moo() {
            this.mowed = true; // mootable state: don't do that at home
            return 'moo!';
        };
    }


测试用例如下：

    // cow-test.js
    casper.test.begin('Cow can moo', 2, function suite(test) {
        var cow = new Cow();
        test.assertEquals(cow.moo(), 'moo!');
        test.assert(cow.mowed);
        test.done();
    });


通过`casperjs test`命令运行测试用例：

    $ casperjs test cow-test.js


你应该卡到类似下面的结果：

![http://docs.casperjs.org/en/latest/_images/cow-test-ok.png](http://docs.casperjs.org/en/latest/_images/cow-test-ok.png)


修改，使测试用例失败：

    casper.test.begin('Cow can moo', 2, function suite(test) {
        var cow = new Cow();
        test.assertEquals(cow.moo(), 'BAZINGA!');
        test.assert(cow.mowed);
        test.done();
    });


结果会变成：

![http://docs.casperjs.org/en/latest/_images/cow-test-ko.png](http://docs.casperjs.org/en/latest/_images/cow-test-ko.png)


##浏览器测试


现在我们给Google搜素写一个测试用例：

    // googletesting.js
    casper.test.begin('Google search retrieves 10 or more results', 5, function suite(test) {
        casper.start("http://www.google.fr/", function() {
            test.assertTitle("Google", "google homepage title is the one expected");
            test.assertExists('form[action="/search"]', "main form is found");
            this.fill('form[action="/search"]', {
                q: "casperjs"
            }, true);
        });

        casper.then(function() {
            test.assertTitle("casperjs - Recherche Google", "google title is ok");
            test.assertUrlMatch(/q=casperjs/, "search term has been submitted");
            test.assertEval(function() {
                return __utils__.findAll("h3.r").length >= 10;
            }, "google search for \"casperjs\" retrieves 10 or more results");
        });

        casper.run(function() {
            test.done();
        });
    });


运行测试：

    $ casperjs test googletesting.js

结果大致如下：

![http://docs.casperjs.org/en/latest/_images/testsuiteok.png](http://docs.casperjs.org/en/latest/_images/testsuiteok.png)

##设置测试环境下的casper选项


因为在测试环境下你必须使用预定义的`casper`实例，所以你可以用下面的方式配置casper

    casper.options.optionName = optionValue; // where optionName is obviously the desired option name

    casper.options.clientScripts.push("new-script.js");


##深入用法

`Tester#begin()`接受一个描述测试用例的函数或者对象作为参数，如果是对象，则可以包含`setUp()` and `tearDown()`方法。

    // cow-test.js
    casper.test.begin('Cow can moo', 2, {
        setUp: function(test) {
            this.cow = new Cow();
        },

        tearDown: function(test) {
            this.cow.destroy();
        },

        test: function(test) {
            test.assertEquals(this.cow.moo(), 'moo!');
            test.assert(this.cow.mowed);
            test.done();
        }
    });


##测试命令参数和选项

###参数

`casperjs test`命令会把传递进来的每个参数看做文件或包含测试文件的目录。它会递归的扫描每个目录，寻找`*.js` or `*.coffee`文件，并把它们加入到堆栈中。

警告：在写测试用例时，有两点需要特别注意：
	
- 不能创建新的Casper实例
- 在一套测试用例（或一个文件）执行的最后必须调用`Tester.done()`


###选项

选项都用双横线`--`做前缀：

- `--xunit=<filename>`：以Xunit XML文件格式导出测试结果
- `--direct` 或者 `--verbose`：把log信息直接输出到控制台
- `--log-level=<logLevel>`：设置log level
- `--auto-exit=no`：阻止test runner在执行完所有测试用例后自动退出；从而允许你执行一些额外的操作，你需要手动调用`exit`:

    	// $ casperjs test --auto-exit=no
    	casper.test.on("exit", function() {
      	someTediousAsyncProcess(function() {
        	casper.exit();
      		});
    	});



1.0版本新增：

- `--includes=foo.js,bar.js`:会在每个测试文件执行前引入`foo.js`和`bar.js`;
- `--pre=pre-test.js`:会在执行整个测试套件之前加入`pre-test.js`文件包含的测试;
- `--post=post-test.js`:会在执行完整个测试套件之后加入`post-test.js`文件包含的测试;
- `--fail-fast`:当有一个错误出现时立即结束当前测试套件;
- `--concise`:会创建一个更加简洁的测试结果;
- `--no-colors`:输出结果非彩色


命令实例：


    $ casperjs test --includes=foo.js,bar.js \
                    --pre=pre-test.js \
                    --post=post-test.js \
                    --direct \
                    --log-level=debug \
                    --fail-fast \
                    test1.js test2.js /path/to/some/test/dir

   
注意：
`--direct`在1.1版本标记为deprecated，用`--verbose`替代。


##导出XUnit格式的结果


CasperJS可以把测试结果保存成XUnit XML文件格式，可以匹配CI工具，例如[Jenkins](http://jenkins-ci.org/)。通过`--xunit`选项来做：


    $ casperjs test googletesting.js --xunit=log.xml


你会得到类似下面的结果：


    <?xml version="1.0" encoding="UTF-8"?>
    <testsuites duration="1.249">
        <testsuite errors="0" failures="0" name="Google search retrieves 10 or more results" package="googletesting" tests="5" time="1.249" timestamp="2012-12-30T21:27:26.320Z">
            <testcase classname="googletesting" name="google homepage title is the one expected" time="0.813"/>
            <testcase classname="googletesting" name="main form is found" time="0.002"/>
            <testcase classname="googletesting" name="google title is ok" time="0.416"/>
            <testcase classname="googletesting" name="search term has been submitted" time="0.017"/>
            <testcase classname="googletesting" name="google search for &quot;casperjs&quot; retrieves 10 or more results" time="0.001"/>
            <system-out/>
        </testsuite>
    </testsuites>


你可以通过传递一个对象给`casper.test.fail()`来自定义`name`属性的值，例如：


    casper.test.fail('google search for "casperjs" retrieves 10 or more results', {name: 'result count is 10+'});

 输出结果：
 
    <?xml version="1.0" encoding="UTF-8"?>
    <testsuites duration="1.249">
        <testsuite errors="0" failures="0" name="Google search retrieves 10 or more results" package="googletesting" tests="5" time="1.249" timestamp="2012-12-30T21:27:26.320Z">
            <testcase classname="googletesting" name="google homepage title is the one expected" time="0.813"/>
            <testcase classname="googletesting" name="main form is found" time="0.002"/>
            <testcase classname="googletesting" name="google title is ok" time="0.416"/>
            <testcase classname="googletesting" name="search term has been submitted" time="0.017"/>
            <testcase classname="googletesting" name="results count is 10+" time="0.001"/>
                <failure type="fail">google search for "casperjs" retrieves 10 or more results</failure>
            <system-out/>
        </testsuite>
    </testsuites>

##CasperJS自测


CasperJS 包含自己的单元测试和功能测试套件，放在test子文件夹中。用下面的命令执行：


    $ casperjs selftest

   
注意：
运行CasperJS自身的测试套件，是发现bug很好地方式。如果测试失败可以[提交issue](https://github.com/n1k0/casperjs/issues/new)



##扩展Casper测试


    $ casperjs test [path]

上面的命令是下面命令的缩写：


    $ casperjs /path/to/casperjs/tests/run.js [path]


所以如果你想扩展Casper的测试能力，最好的方式就是写自己的runner，并扩展这里的casper对象实例。


提示：默认的runner代码在[run.js](https://github.com/n1k0/casperjs/blob/master/tests/run.js)