---
layout: post
category : javascript
title: "Ajax 总结"
tagline: "Supporting tagline"
tags : []
---

Ajax 是 Asynchronus Javascript + XML 的缩写。Ajax 技术的核心是 XMLHttpRequest 对象（简称 XHR），由微软最先引入，后来其它浏览器也提供了 XHR 的实现。XHR 提供了以异步方式向服务器发送请求和解析服务器响应的接口。

规范的一些历史：https://xhr.spec.whatwg.org/#specification-history

> The XMLHttpRequest object was initially defined as part of the WHATWG’s HTML effort. (Based on Microsoft’s implementation many years prior.) It moved to the W3C in 2006. Extensions (e.g. progress events and cross-origin requests) to XMLHttpRequest were developed in a separate draft (XMLHttpRequest Level 2) until end of 2011, at which point the two drafts were merged and XMLHttpRequest became a single entity again from a standards perspective. End of 2012 it moved back to the WHATWG.



## XHR 的使用

XHR 有两种用法：
1. 同步方式：发出请求后，JavaScript 代码会等到服务器响应之后再继续执行。
2. 异步方式：发出请求后，JavaScript 继续执行而不必等待响应。

多数情况下，我们都是采用异步的方式。下面是异步 XHR 最基本的使用流程：

1. 通过 `XMLHttpRequest()` 构造函数来创建 xhr 实例
2. 监听 `onreadystatechange` 事件。XHR 对象的 `readyState` 属性表示请求/响应过程的当前活动状态，可取值为 0 到 4 （后面会说）。`readyState` 属性值每一次变化都会触发 `readystatechange` 事件。通常只需关心值为 4 的阶段。为了保证兼容性，必须在调用 `open()` 方法之前绑定 `onreadystatechange` 事件处理程序。
3. 调用 `open()` 方法创建请求。`open()` 方法接收三个参数：待发送请求的类型（`'get'`、`'post'` 等）；请求的 URL；是否异步发送请求。注意：`open()` 方法并不会真正发送请求，只是启动一个请求以备发送
4. 调用 `send()` 方法发送请求。参数是作为请求主体发送的数据，当请求方法是 GET 或者 HEAD 时，参数被忽略。

一个简单的例子：

```javascript
var xhr = new XMLHttpRequest();
xhr.onreadystatechange = function(){
    if(xhr.readyState==4){
        if(xhr.status>=200 && xhr.status<300 || xhr.status==304){
            alert(xhr.responseText)
        }else{
            alert('fail')
        }
    }
}
xhr.open('get', 'api.php', 'true');
xhr.send();
```

XHR 对象的 `readyState` 属性可取值及其对应状态：

- 0：未初始化。未调用 `open` 方法
- 1：启动。已调用 `open` 方法，未调用 `send` 方法
- 2：发送。已调用 `send` 方法，未接到响应
- 3：接收。已接到部分响应数据
- 4：完成。已经接受到全部响应，可以使用

收到响应后，响应的数据会自动填充 XHR 对象的属性，包括：

- `responseText`：作为响应主体返回的文本
- `responseXML`：如果响应内容类型是 'text/xml' 或 'application/xml'，这个属性将保存响应数据的 XML DOM 文档。
- `status`：响应的 HTTP 状态
- `statusText`：HTTP 状态说明

### send 方法
内部操作：
https://xhr.spec.whatwg.org/#the-send()-method

## HTTP 头部信息

### 请求头
在 `open()` 方法之后，`send()` 方法之前，可以调用 `setRequestHeader()` 方法设置自定义请求头部信息。例如：

```javascript
xhr.setRequestHeader("myheader","value");
```

除了可以设置自定义请求头，还可以重写某些标准请求头，例如：'Content-Type'，出于安全考虑，浏览器会禁止重写某些请求头，例如：'Host'、'cookie' 等，[参见这里](https://xhr.spec.whatwg.org/#dom-xmlhttprequest-setrequestheader)。试图重写这些请求头，会触发异常：

```javascript
Refused to set unsafe header "Host"
```


### 响应头
XHR 对象的 `getResponseHeader()` 和 `getAllResponseHeaders()` 方法可以获取响应头。

## XMLHttpRequest 2 

XMLHttpRequest 2 级规范进一步扩展了 XHR。包括：

- FormData
- timeout 属性

### FromData
现代网页经常用到的一项功能就是表单数据的序列化。XMLHttpRequest 2 级规范为此定义了 `FormData` 类型。

给 FormData 填充数据的两种方式:

```javascript
var data = new FormData();
data.append('name','value');

// 或者直接使用表单元素
var data = new FormData(document.forms[0])
```
然后就可以将 FormData 传给 `send` 方法。

### timeout 属性

### 发送和接收二进制数据

- 发送

XMLHttpRequest 对象的 send 方法，可以接受 ArrayBuffer, Blob, File 对象。

- 接收

XMLHttpRequest 对象的 responseType 属性可以用来改变期望服务端返回的响应类型。可选值：空字符串（默认值）、
"arraybuffer", "blob", "document", "json", 和 "text"。response 属性包含 entity body according to responseType, as an ArrayBuffer, Blob, Document, JSON, or string. 

[https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/Sending_and_Receiving_Binary_Data](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/Sending_and_Receiving_Binary_Data)

## Progress Events

[https://www.w3.org/TR/progress-events/](https://www.w3.org/TR/progress-events/)

> The Progress Events specification defines an event interface that can be used for measuring progress。

XHR 提供了以下几个进度事件

- loadstart：在接收到响应数据的第一个字节时触发
- progress：在接收响应期间持续不断地触发
- error：在请求发生错误时触发
- abort：在因为调用 `abort()` 方法而终止连接时触发
- load：在接收到完整响应数据时触发
- loadend：在通信完成或者触发 error、abort 或 load 事件后触发

### load 事件
有了 load 事件，就无需再通过监听 `readystatechange` 事件来获取响应了。上面的例子可以优化为：

```javascript
var xhr = new XMLHttpRequest();
xhr.onload = function(){
    var status = xhr.status;
    if(status >= 200 && status < 300 || status == 304){
        alert(xhr.responseText)
    }else{
        alert('request was failed: '+ status);
    }
}
xhr.open('get','api.php',true);
xhr.send();
```

### progress 事件
progress 事件会在浏览器接收数据期间周期性的触发。onprogress 事件处理函数会接收一个 event 对象，其中 `target` 属性是 XHR 对象，但包含着三个额外的属性：

- lengthComputable：布尔值，表示进度信息是否可用
- position：已接收的字节数
- totalSize：根据 Content-Length 响应头确定的预期字节数

## 跨域 Ajax

由于同源策略的限制，XHR 对象只能访问与包含它的页面位于同一个域中的资源。但是，实现合理的跨域请求对开发浏览器应用程序也是至关重要的。于是 W3C 提出了 CORS（cross-origin resource sharing）。CORS 的基本思路是：使用自定义的 HTTP 头让浏览器与服务器进行沟通，从而决定请求或响应是否应该成功。

ajax 会自动带上 cookie ，对于跨站请求，浏览器是不会发送凭证信息的。但如果将 XMLHttpRequest 的一个特殊标志位设置为 true，浏览器就将允许该请求的发送。

```javascript
xhr.withCredentials = true;
```
但是，如果服务器端的响应中，如果没有返回 `Access-Control-Allow-Credentials: true`的响应头，那么浏览器将不会把响应结果传递给发出请求的脚本程序，以保证信息的安全。

https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/withCredentials

withCredentials 还用来指示是否忽略响应中的 Cookie。

注意：跨域并非浏览器限制了发起跨站请求，而是跨站请求可以正常发起，但是返回结果被浏览器拦截了。最好的例子是 CSRF 跨站攻击原理，请求是发送到了后端服务。

还要注意的是：有些浏览器不允许从 HTTPS 的域跨域访问 HTTP，比如 Chrome 和 Firefox，这些浏览器在请求还未发出的时候就会拦截请求。

在 Chrome 中， https://a.com ajax 请求 http://a.com，会在控制台输出

> Mixed Content: The page at 'https://a.com' was loaded over HTTPS, but requested an insecure XMLHttpRequest endpoint 'http://a.com'. This request has been blocked; the content must be served over HTTPS.

同时，在 Network 面板，对应请求 status 标记为：(blocked:mixed-content)	

https://developer.mozilla.org/en-US/docs/Web/Security/Mixed_content

http://javascript.ruanyifeng.com/bom/ajax.html#toc11