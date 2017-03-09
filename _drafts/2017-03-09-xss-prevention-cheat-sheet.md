---
layout: post
category : security
title: "XSS 防御备忘录"
tagline: "Supporting tagline"
tags : [xss]
---

反射型 XSS 和存储型 XSS 都可以通过在服务端执行合理的编码（encoding/escaping）、验证（validation）来防御。本文主要关注反射型 XSS 和存储型 XSS 的防御。DOM XSS 的防御参见：[DOM based XSS Prevention cheat sheet](https://www.owasp.org/index.php/DOM_based_XSS_Prevention_Cheat_Sheet)

## A Positive XSS Prevention Model

本文把 HTML 页面看做一个模板，模板中有些位置允许开发者放置不可信数据，而在其他位置放置不可信数据是不允许的。也就是说，这是一个白名单模型，除了特别声明允许之外的的全部都禁止。

## 你需要一个安全的编码库

- .net [Microsoft Anti-Cross Site Scripting Library](http://wpl.codeplex.com/)
- ASP.net 内置的 [ValidateRequest](http://msdn.microsoft.com/en-us/library/ms972969.aspx#securitybarriers_topic6) 函数提供了有限的净化功能
- Java [OWASP ESAPI](https://www.owasp.org/index.php/ESAPI)

## XSS 防御规则


### RULE #0 在那些允许的位置之外，不要插入不可信的数据

第一条规则就是 deny all —— 除了 Rule #1 至 Rule #5 那些规定的位置之外，不要在其它位置插入不可信数据。因为 HTML 中有很多特殊的上下文，在这些上下文中转义规则会变得非常复杂。

```
<script>...NEVER PUT UNTRUSTED DATA HERE...</script>   不要直接插入到 script 标签中
 
<!--...NEVER PUT UNTRUSTED DATA HERE...-->             不要插入到 HTML 注释中
 
<div ...NEVER PUT UNTRUSTED DATA HERE...=test />       不要放在属性名的位置
 
<NEVER PUT UNTRUSTED DATA HERE... href="/test" />   不要放在标签名的位置
 
<style>...NEVER PUT UNTRUSTED DATA HERE...</style>   不要直接放在 CSS 中
```

Most importantly, never accept actual JavaScript code from an untrusted source and then run it. For example, a parameter named "callback" that contains a JavaScript code snippet. No amount of escaping can fix that.

### RULE #1 在将不可信数据插入到 HTML 元素内容之前进行 HTML 转义
插入到普通标签（例如：div、p、b、td 等）之前，要进行 HTML 转义

```
<body>...ESCAPE UNTRUSTED DATA BEFORE PUTTING HERE...</body>
 
<div>...ESCAPE UNTRUSTED DATA BEFORE PUTTING HERE...</div>
 
any other normal HTML elements
```

需要转义以下字符：

```
& --> &amp;
< --> &lt;
> --> &gt;
" --> &quot;
' --> &#x27;     &apos; not recommended because its not in the HTML spec (See: section 24.4.1) &apos; is in the XML and XHTML specs.
/ --> &#x2F;     forward slash is included as it helps end an HTML entity
```

### RULE #2 在将不可信数据插入到 HTML 普通属性值之前要进行 Attribute 转义

普通属性包括：width、name、value 等等，不包括复杂属性，例如：href,src,style 或其它任何事件处理器（onmouseover 等）。事件处理器应该遵守 RULE #3

```
<div attr=...ESCAPE UNTRUSTED DATA BEFORE PUTTING HERE...>content</div>     没有引号
 
<div attr='...ESCAPE UNTRUSTED DATA BEFORE PUTTING HERE...'>content</div>   单引号
 
<div attr="...ESCAPE UNTRUSTED DATA BEFORE PUTTING HERE...">content</div>   双引号
```

用 `&#xHH` 格式（命名实体也可以）转义所有 ASCII 值小于 256 的除字母数字之外的所有字符，来防止逃逸出属性。需要转义这么多字符，是因为开发者经常不使用引号。使用引号的属性只能用对应的引号才能逃逸，而不使用引号，很多字符都可以用来逃逸，包括：[space] % * + , - / ; < = > ^ 和 |。

### RULE #3 在将不可信数据插入到 JavaScript data value 之前要进行 JavaScript 转义
规则三主要针对动态生成的 JavaScript 代码 —— 包括 script 块和事件处理器属性。放置不可信数据唯一安全的地方就是引号包裹的 data value。而在其它 JavaScript 上下文插入不可信数据是很危险的，因为利用一些字符可以很容易切换到可执行上下文，这包括但不限于：semi-colon,equals,space,plus 等等很多。

```
<script>alert('...ESCAPE UNTRUSTED DATA BEFORE PUTTING HERE...')</script>     inside a quoted string
 
<script>x='...ESCAPE UNTRUSTED DATA BEFORE PUTTING HERE...'</script>          one side of a quoted expression
 
<div onmouseover="x='...ESCAPE UNTRUSTED DATA BEFORE PUTTING HERE...'"</div>  inside quoted event handler
```

需要注意的是，有一些 JavaScript 函数，使用不可信数据时，即使进行了 JavaScript 转义，也是不安全的。例如：

```
<script>
 window.setInterval('...EVEN IF YOU ESCAPE UNTRUSTED DATA YOU ARE XSSED HERE...');
</script>
```

用 `\xHH` 格式转义所有 ASCII 值小于 256 的除字母数字之外的所有字符，来防止从 data value 逃逸到 script context 或是其他属性。

不要使用转义的快捷写法，例如：`\"`，因为引号可能会先被 HTML 属性解析器匹配到。而且快捷写法容易遭到 escape-the-escape 攻击，如果攻击者发送 `\"`，会被转义成 `\\"`，从而引号生效。

### RULE #3.1 

在 Web 2.0 的世界，经常需要应用程序动态生成一些数据。一种策略是，发送 Ajax 请求来获取数据。但出于性能考虑，经常采用的另一种策略是在初始页面中嵌入一个 JSON 块。

确保响应头 Content-Type 的值是 application/json 而不是 text/html。

错误的 HTTP 响应

```
HTTP/1.1 200
Date: Wed, 06 Feb 2013 10:28:54 GMT
Server: Microsoft-IIS/7.5....
Content-Type: text/html; charset=utf-8 <-- bad
....
Content-Length: 373
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
{"Message":"No HTTP resource was found that matches the request URI 'dev.net.ie/api/pay/.html?HouseNumber=9&AddressLine
   =The+Gardens<script>alert(1)</script>&AddressLine2=foxlodge+woods&TownName=Meath'.","MessageDetail":"No type was found
   that matches the controller named 'pay'."}   <-- this script will pop!!
```

正确的 HTTP 响应

```
HTTP/1.1 200
Date: Wed, 06 Feb 2013 10:28:54 GMT
Server: Microsoft-IIS/7.5....
Content-Type: application/json; charset=utf-8 <--good
.....
.....
```

一个常见的反模式是：

```
<script>
     var initData = <%= data.to_json %>; // Do NOT do this without encoding the data with one of the techniques listed below.
</script>
```

JSON 实体编码

HTML 实体编码

An alternative to escaping and unescaping JSON directly in JavaScript, is to normalize JSON server-side by converting '<' to '\u003c' before delivering it to the browser.

### RULE #4 在将不可信数据插入到 HTML style 属性值之前要进行 CSS 转义和严格的校验

```
<style>selector { property : ...ESCAPE UNTRUSTED DATA BEFORE PUTTING HERE...; } </style>     property value

<style>selector { property : "...ESCAPE UNTRUSTED DATA BEFORE PUTTING HERE..."; } </style>   property value

<span style="property : ...ESCAPE UNTRUSTED DATA BEFORE PUTTING HERE...">text</span>       property value
```

注意，在有些 CSS 上下文，即使你进行了合理的 CSS 转义，也不能保证安全。你必须保证 URL 以 http 开始而不是 JavaScript，同时不能以 expression 开始

```
{ background-url : "javascript:alert(1)"; }  // and all other URLs
{ text-size: "expression(alert('XSS'))"; }   // only in IE
```
用 `\HH` 格式转义所有 ASCII 值小于 256 的除字母数字之外的所有字符。不要使用转义快捷方式。

### RULE #5 在将不可信数据插入到 HTML URL 参数值之前要进行 URL 转义

HTTP GET 参数值

```
 <a href="http://www.somesite.com?test=...ESCAPE UNTRUSTED DATA BEFORE PUTTING HERE...">link</a >   
```
用 `%HH` 格式转义所有 ASCII 值小于 256 的除字母数字之外的所有字符。Including untrusted data in data: URLs should not be allowed as there is no good way to disable attacks with escaping to prevent switching out of the URL. 

Note that entity encoding is useless in this context.

警告：Do not encode complete or relative URL's with URL encoding! 如果要把不可信数据插入到 href，src 或其它 URL-based 属性，要确保没有指向其它协议，尤其是 JavaScript。URL's should then be encoded based on the context of display like any other piece of data.

### RULE #6 用专门的库来净化 HTML 标签

如果你的应用需要处理标记 —— 不可信数据可以包含 HTML —— 这样验证起来很困难。

- .net https://github.com/mganss/HtmlSanitizer
- java https://github.com/OWASP/java-html-sanitizer
- Ruby on Rails http://api.rubyonrails.org/classes/ActionView/Helpers/SanitizeHelper.html
- PHP http://htmlpurifier.org/
- javascript/node.js https://github.com/ecto/bleach
- python https://pypi.python.org/pypi/bleach

### RULE #7 防御 DOM XSS

### Bonus Rule #1 使用 HTTPOnly cookie

### Bonus Rule #2 实现 CSP

### Bouns Rule #3 使用自动转义模板系统
许多 Web 应用框架提供了 automatic contextual escaping 功能，例如：[AngularJS strict contextual escaping](https://docs.angularjs.org/api/ng/service/$sce) 和 [Go templates
](https://golang.org/pkg/html/template/)

### Bonus Rule #4 使用 X-XSS-Protection 响应头

## XSS 防御规则总结
|数据类型|上下文|代码示例|防御规则|
|:--|:---|:--|:--|
|字符串|HTML Body|`<span>UNTRUSTED DATA</span>`|HTML 编码|
|字符|安全的 HTML 属性|`<input type="text" name="fname" value="UNTRUSTED DATA">`||
|字符串|Get 参数|`<a href="/site/search?value=UNTRUSTED DATA">clickme</a>`|URL 编码|
|字符串|src 或 href 属性|`<a href="UNTRUSTED URL">clickme</a>`<br>`<iframe src="UNTRUSTED URL" />`||
|字符串|CSS 值|`<div style="width: UNTRUSTED DATA;">Selection</div>`||
|字符串|JavaScript 变量|`<script>var currentValue='UNTRUSTED DATA';</script>`<br>`<script>someFunction('UNTRUSTED DATA');</script>`||
|HTML|HTML body|`<div>UNTRUSTED HTML</div>`|HTML 校验|
|字符串|DOM XSS|`<script>document.write("UNTRUSTED INPUT: " + document.location.hash);<script/>`||

安全的 HTML 属性包括：align, alink, alt, bgcolor, border, cellpadding, cellspacing, class, color, cols, colspan, coords, dir, face, height, hspace, ismap, lang, marginheight, marginwidth, multiple, nohref, noresize, noshade, nowrap, ref, rel, rev, rows, rowspan, scrolling, shape, span, summary, tabindex, title, usemap, valign, value, vlink, vspace, width

## 输出编码规则总结

|编码类型|编码机制|
|:--:|:--|
|HTML 编码|`&` 编码为 `&amp;` <br>`<` 编码为 `&lt;`<br> `>` 编码为 `&gt;`<br> `"` 编码为 `&quot;`<br> `'` 编码为 `&#x27;`<br> `/` 编码为 `&#x2F;`<br>|
|HTML 属性编码|除了字母和数字，用 HTML 实体编码 `&#xHH;` 格式编码所有字符，包括空格|
|URL 编码|标准百分号编码。URL 编码只应该用来编码参数值，而不是整个 URL 或是 URL path 片段|
|JavaScript 编码|除了字母和数字，用 unicode 编码格式 `\uXXXX` 编码所有字符|
|CSS hex 编码||









