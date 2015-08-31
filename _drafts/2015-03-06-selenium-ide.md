---
layout: post
category : lessons
title: "selenium-IDE"
tagline: ""
tags : [selenium]
---

##简介
##安装
##打开IDE
##IDE功能
###菜单栏
test suite 是一套test case。

###工具条
###test case 面板
###log/reference/ui-element/rollup
##创建Test case
###录制
需要注意的是：

 - 要想记录`type`命令需要在页面某处点击一下
 - 点击一个连接通常被录制为`click`，通常情况下你需要将其变成`clickAndWait`，这是为了暂停测试用例直到新页面加载完成。否则，测试用例会在页面加载完成之前继续运行，这会导致不可预料的错误。

###添加断言和验证

###编辑
##运行Test case
双击某条命令，会单独运行该命令。
##使用Base URL 在不同域名下运行测试用例
##Selenium 命令——"Selenese"
通过Selenese你可以测试标签是否存在，测试指定内容，测试损坏的链接，输入框，下拉列表，提交表单，表格数据。还有窗口大小，鼠标位置，alert，Ajax,弹框，事件处理等等。

selenium命令可以分为三类

 - Actions。如果一个Action 失败，或者有错误，当前测试用例会失败。
 - Accessors。检查应用的状态，并将结果存到变量中。例如：`storeTitle`
 - Assertions。Assertions可以在三种模式下使用：assert,verify和waitFor。例如你可以`assertText`,`verifyText`,`waitForText`.

##脚本语法
##常用命令

 - open
 - click/clickAndWait
 - verifyTitle/assertTitle
 - verifyTextPresent
 - verifyElementPresent
 - verifyText 验证文本及对应的标签存在于网页中
 - verifyTable
 - waitForPageToLoad 暂停执行直到新页面加载完。会被clickAndWait自动调用
 - waitForElementPresent

##验证页面元素

##Assertion 还是 Verification
选择Assert还是verify可以归结为如何方便的管理失败用例。

一个Assert会使当前测试失败并终止当前test case，而一个verify会使当前测试失败但会继续执行当前test case。

最好是以合乎逻辑的顺序分组你的测试命令，每一组以一个assert开始，后面跟着一个或多个verify。

###verify TextPresent
`TextPresent`命令用来验证指定文本是否在页面中存在。它接受一个参数：要验证的文本。当你只关心文本是否出现时可以用这条命令，但当你关心文本出现的位置时，不要用这条命令。

###verifyElementPresent
`verifyElementPresent`用来验证指定标签是否在页面中存在。只验证标签，而不是其内容。通常用来检查图片是否存在。

###verifyText
`verifyText`用来验证标签以及内容是否存在。

##定位元素
location strategy 为locatorType=location。locatorType 有好多种，如下：
###根据Identifier定位

这是最常用的也是默认使用的locatorType。根据ID定位时，会返回第一个匹配的元素。如果没有匹配的，那么返回第一个name属性值匹配该id的元素.

示例：
```
<html>
 <body>
  <form id="loginForm">
   <input name="username" type="text" />
   <input name="password" type="password" />
   <input name="continue" type="submit" value="Login" />
  </form>
 </body>
<html>
```

identifier=loginForm (3)
identifier=password (5)
identifier=continue (6)
continue (6)
由于identifier类型的locator是默认使用的，所以前三个的‘identifier=’不是必须的，可以写成第四个


####根据ID定位
id=loginForm
####根据name定位

```
 <html>
  <body>
   <form id="loginForm">
    <input name="username" type="text" />
    <input name="password" type="password" />
    <input name="continue" type="submit" value="Login" />
    <input name="continue" type="button" value="Clear" />
   </form>
 </body>
 <html>
```

当有多个元素拥有相同name时，你可以使用过滤器，默认的过滤器是value(匹配value值)

name=username (4)
name=continue value=Clear (7)
name=continue Clear (7)
name=continue type=button (7)

注意：上面提到的三种定位方式都不会涉及到标签层级结构。而xpath和Dom 两种定位方式会涉及到标签的层级

####根据Xpath定位
因为只有xpath locator以'//'开头，所以当使用xpath locator时，不需要xpath=

有用的插件

 - xpath checker
 - firebug

####根据链接内容定位超链接
如果两个链接内容相同，则返回第一个。
```
<html>
 <body>
  <p>Are you sure you want to do this?</p>
  <a href="continue.html">Continue</a>
  <a href="cancel.html">Cancel</a>
</body>
<html>
```
link=Continue (4)
link=Cancel (5)
 
####根据DOM定位
因为只有Dom locator以'document'开始，所以当使用Dom locator时，不需要dom=

```
<html>
  <body>
   <form id="loginForm">
    <input name="username" type="text" />
    <input name="password" type="password" />
    <input name="continue" type="submit" value="Login" />
    <input name="continue" type="button" value="Clear" />
   </form>
 </body>
 <html>
```

dom=document.getElementById('loginForm') (3)
dom=document.forms['loginForm'] (3)
dom=document.forms[0] (3)
document.forms[0].username (4)
document.forms[0].elements['username'] (4)
document.forms[0].elements[0] (4)
document.forms[0].elements[3] (7)
####根据css定位
大多数selenium用户建议使用css locator，因为它比xpath快。

##匹配文本模式
##'AndWait'命令
需要注意的是：如果你用了一个AndWait命令，而对应的action并没有触发navigation/refresh，你的测试会失败。这是因为当没有任何navigation或refresh的时候，selenium会触发AndWait的timeout，然后抛出一个timeout exception
##在ajax应用中使用waitFor命令
在ajax应用中，页面不会刷新。使用`andWait`命令就不起作用了，因为页面没有刷新。通过暂停测试运行一定时间也不是最好的方法，因为响应的时间可长可短，会导致测试失败。最好的方法应该是在一定时间内等待需要的元素，当发现需要的元素时继续执行测试。

这就需要`waitFor`命令，比如`waitForElementPresent`或`waitForVisible`,这些命令每秒检查一次需要的状态，一旦状态满足，就继续执行。

##流程控制
Selenese本身是不支持条件语句和循环语句的。当你需要流程控制的时候，有三种选择：

 - 列表项

##保存命令、selenium变量
`store`是最基本的用来保存值的命令。
使用的时候用${变量名}
其它常用store命令
storeElementPresent 对应verifyElementPresent。它保存一个布尔值，根据对应的元素是否存在。
storeText对应verifyText
storeEval 接受一段脚本作为参数。把脚本的运行结果保存到变量里。

##