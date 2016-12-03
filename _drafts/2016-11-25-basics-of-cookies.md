---
layout: post
category : http
title: "Cookie 基础"
tagline: "Supporting tagline"
tags : [cookie]
---


## cookie 的起源

HTTP 最初是一个无状态的请求/响应协议。也就是说服务器没办法知道两个请求是否来自同一个浏览器。为了解决这一问题，开发者们引入了一些用户识别机制：

- HTTP 认证，用认证方式来识别用户
- 胖 URL，在 URL 中嵌入标识信息
- Cookie
- 等

其中，cookie 是当前识别用户，实现持久会话的最好方式。cookie 最初是由网景公司开发的，但现在所有主要的浏览器都支持它。

## cookie 是什么？

简单地说，cookie 就是浏览器保存在用户计算机上的一小段文本。

网页或者服务器指示浏览器保存一些状态信息到 cookie，在后续请求中，浏览器会根据一定的规则将 cookie 发送到服务器，服务器用这些状态信息来识别用户。

cookie 常见的用途有：

- 会话管理（用户登录、购物车）
- 个性化（保存一些用户偏好）
- 跟踪（分析用户行为）

## 新建 cookie

服务器通过使用 HTTP 响应头 `Set-Cookie` 来设置 cookie：

```
Set-Cookie: key=value; path=path; domain=domain; max-age=max-age-in-seconds; expires=date-in-GMTString-format; secure; HttpOnly
```

`Set-Cookie` 声明的 cookie 属性要用分号和空格分割。每一个属性都定义了一条规则，用来判断何时将该 cookie 发送回服务器。属性都是可选的。



浏览器使用请求头 `Cookie` 发送 cookie：

```
Cookie: key1=value1; key2=value2
```

注意：`Set-Cookie` 设置的 cookie 属性是给浏览器用的，所以不会被发送回服务器，一旦设置，就没办法再获取。

### cookie 编码

规范并没有明确要使用何种编码，大部分实现都是采用 URL 编码。

### 过期时间

跟过期时间相关的属性有两个：

- expires
- max-age

#### expires 

expires 用来指示 cookie 何时过期。格式为：

```
Wdy, DD-Mon-YYYY HH:MM:SS GMT
```

如果将 expires 的值设置为过去的某个时间，那么 cookie 立即失效。

#### max-age

max-age 用来指示 cookie 在多少秒后过期。可以作为对 expires 的补充。如果 max-age 和 expires 同时出现，max-age 优先级更高。

根据过期时间的设置，可以将 cookie 分为两类：

- 会话 cookie：Set-Cookie 时，没有指定过期时间。cookie 仅在一个会话中有效，浏览器关闭，会话结束，cookie 就失效。
- 持久 cookie：Set-Cookie 时，指定过期时间为以后的某个时间。

    
### domain

domain 用来指示 cookie 会被发送到哪些域名。

根据 domain 的设置，可以将 cookie 分为：

- host-only cookie：Set-Cookie 时不包含 domain 项，或者 domain 值为空。
- domain cookie：Set-Cookie 时设置了 domain 值，且值合法

在 chrome 和 Firefox 中，为了和 host-only cookie 进行区分，domain cookie 的 domain 值都以 `.` 打头。

为了便于理解，假设有两个域名 play.com 和 www.play.com。

#### 设置 cookie 时：

- 如果省略 domain 项，或者 domain 值为空（即 host-only cookie），则 domain 与 Set-Cookie 的页面的主机名（host name）相同。即 play.com 设置的 cookie 的 domain 值为 `play.com`。www.play.com 设置的 cookie 的 domain 值为 `www.play.com`
- 如果不省略 domain 项，domain 的值只能是 Set-Cookie 的页面的主机名的一部分。
	- play.com 不能设置 google.com 的 cookie，www.play.com 不能设置 dev.play.com 的 cookie。
	- 子域可以将 domain 值设置为父域，反过来不行。即 www.play.com 可以将 cookie 的 domain 设置为 `play.com`，但是 play.com 不能将 cookie 的 domain 值设置为 `www.play.com`。
	- domain 值不能是顶级域，例如：.com
	- 非法的 domain 会被丢弃
	

#### 发送 cookie 时

判断是否发送某个 cookie，要看是否域名匹配，分两种情况：

- host-only cookie：cookie 的 domain 值与请求 URL 的主机名完全一致时，才发送
- domain cookie：cookie 的 domain 值是请求 URL 的主机名的后缀，或是 domain 值去掉前置点后，与请求 URL 的主机名一致时，才发送

举例说明：

- 访问 www.play.com 时，会带上 domain 值为 `.play.com` 、`.www.play.com` 和 `www.play.com` 的 cookie，但是不会带上 domain 值为 `play.com` 的 cookie
- 访问 play.com 时，会带上 domain 值为 `play.com` 和 `.play.com` 的 cookie

相关代码：

- CanonicalCookie::IsDomainMatch [https://chromium.googlesource.com/chromium/src/+/master/net/cookies/canonical_cookie.cc#348](https://chromium.googlesource.com/chromium/src/+/master/net/cookies/canonical_cookie.cc#348)
- GetCookieDomainWithString [https://chromium.googlesource.com/chromium/src/+/master/net/cookies/cookie_util.cc#110](https://chromium.googlesource.com/chromium/src/+/master/net/cookies/cookie_util.cc#110)

### path

path 选项可以将 cookie 的作用域限定为某个路径范围。

#### 设置 cookie 时

假设某个响应头包含以下内容：

```
Set-Cookie: key=value; path=pathValue;
```

浏览器会做如下处理：

- 如果 pathValue 为空或是第一个字符不是 %x2F ("/")，则将 path 值设为默认路径（Set-Cookie 的页面路径）。
- 否则将 path 值设置为 pathValue。

#### 发送 cookie 时

只有在 domain 已经匹配的情况下，才会进行路径匹配。

当满足下列条件之一时，我们就说 cookie-path 路径匹配 request-path

- cookie-path 和 request-path 相同
- cookie-path 是 request-path 的前缀，并且 cookie-path 的最后一个字符是 %x2F ("/")
- cookie-path 是 request-path 的前缀，并且 the first character of the request-path that is not included in the cookie-path is a %x2F ("/") character.（换句话说，request-path 去掉 cookie-path 这个前缀之后，剩余的字符第一个字符是 '/'）

相关代码：

- CanonicalCookie::IsOnPath [https://chromium.googlesource.com/chromium/src/+/master/net/cookies/canonical_cookie.cc#313](https://chromium.googlesource.com/chromium/src/+/master/net/cookies/canonical_cookie.cc#313)

### secure

- 注意：它是个 flag ，没有值
- 只有当请求通过 SSL 和 HTTTPS 发送时，才发送 secure cookie
- 永远不要用 cookie 保存敏感信息，因为整个机制在本质上并不安全
- 默认，通过 HTTTPS 设置的 cookie 会自动设置为 secure

### HttpOnly

- 微软在 IE6 SP1 中引入了 HTTP-only cookie。用来告诉浏览器不能通过 `document.cookie` 访问，用来防止通过 XSS 攻击窃取 cookie。
- 只需在 Set-Cookie 时，加上 HttpOnly flag
- 不能通过 JavaScript 设置 HTTP-only cookie


## cookie 维护与生命周期

- 一个 cookie 由 name, domain, path, 和 secure 四个 flag 标记，要想修改 cookie 的值，Set-Cookie 的选项必须和之前的一致，否则就变成了设置一个新的 cookie
- Cookie 头可以包含相同 name 的 cookie。同名 cookie 按照 domain-path-secure 元组，越明确的排在前面
- cookie 的过期时间会和运行浏览器的计算机时间进行对比

### cookie 自动移除
有一些原因会触发浏览器自动移除 cookie：

- 浏览器关闭时，移除 session cookie
- cookie 的过期时间到了
- cookie 容量到了限制点

## cookie 的限制

- 数量限制：一开始规定是每个域名 20个，后来逐渐放开
- 大小限制：4KB


子 cookie：为了绕过数量限制，开发者开始使用子 cookie，即在一个 cookie 中保存多个键值对：

```
name=a=b&c=d&e=f&g=h
```

## JavaScript 中的 cookie

- `document.cookie` 用来创建、操作、删除 cookie
- 设置 cookie 时，格式与 Set-Cookie 一致
	```
document.cookie="name=Nicholas; domain=nczonline.net; path=/";
```
- 设置 `document.cookie` 的值并不会删除页面上所有 cookie，而是新建或者修改对应 cookie
- 通过 JS 设置的 cookie 与通过 Set-Cookie 设置的 cookie 没有区别
- 通过 `document.cookie` 读取 cookie 时，格式与 Cookie 头部一致，多个 cookie 之间用分号和空格分隔
- 通过 `document.cookie` 读取 cookie 时，访问规则（域名匹配、路径匹配、security）与发送到服务端的规则相同。
- 注意：没办法通过 JavaScript 获取 cookie 属性。


## cookie 与缓存

## 相关规范


|标题|描述|位置|
|:--|:--|:--|
|持久客户端状态：HTTP cookies|最初的 Netscape cookie 规范| [http://home.netscape.com/newsref/std/cookie_spec.html](http://web.archive.org/web/20060615215758/wp.netscape.com/newsref/std/cookie_spec.html)|
|RFC 2965: HTTP 状态管理机制|2000 年 10 月的 cookie 标准，废弃了 RFC 2109|[https://www.ietf.org/rfc/rfc2965.txt](https://www.ietf.org/rfc/rfc2965.txt)|
|RFC 6265: HTTP 状态管理机制|废弃了 RFC 2965|[https://tools.ietf.org/html/rfc6265](https://tools.ietf.org/html/rfc6265)|

参考资料：

- 《HTTP 权威指南》P278
- [https://www.nczonline.net/blog/2009/05/05/http-cookies-explained/](https://www.nczonline.net/blog/2009/05/05/http-cookies-explained/)
- [https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)
- [https://www.chromium.org/developers/design-documents/network-stack/cookiemonster](https://www.chromium.org/developers/design-documents/network-stack/cookiemonster)
- [你所不知道的HostOnly Cookie](https://imququ.com/post/host-only-cookie.html)








