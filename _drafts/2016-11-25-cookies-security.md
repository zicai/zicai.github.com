---
layout: post
category : http
title: "Cookie 安全"
tagline: "Supporting tagline"
tags : [cookie]
---

cookie 存在一些安全上的陷阱，突出的有：

- Ambient Authority
	- CSRF 
- 明文传输
- 会话标识
	- 会话固定攻击
- 弱保密性
- 弱完整性
- 依赖 DNS

### 会话劫持
常见会话机制的问题在于，将用户验证交给了单一的数据点-- cookie。再加上，cookie 在网络上是明文传输，可以通过包嗅探获取到。获取到 cookie 之后，攻击者就可以假冒他人进行操作。这种攻击叫做会话劫持（session hijacking），有几种应对措施：

- 最常用的技术就是，只通过 HTTPS 发送 cookie。因为 SSL 会在请求传送到网络之前对其进行加密，这样包嗅探就不能获取 cookie 的值。
- 用随机的方式基于用户信息（用户名、IP 地址、登录时间）生成 session key。这样就增加了重用 session key 的难度。
- 在执行重要操作时（例如：转账）重新验证用户。

### 第三方 cookie

网站可以引用第三方资源（例如： `<link>`、`<script>`、`<object>`、`<iframe>`），第三方响应资源时可以设置自己的 cookie。同时，这些第三方服务器根据接收到的请求头中的 Referer 信息，可以知道用户访问了哪些网站。这样第三方可以用 cookie 标识用户，来跟踪用户访问行为。

### XSS

加载其它域下的脚本会带来一定的安全隐患。虽然说，请求第三方脚本资源时，不会带上页面的 cookie。但是加载完成后的脚本是可以访问页面 cookie 的。页面中所有 JavaScript （不管是页面自身域下，还是从其它域加载）都被认做与页面本身运行在相同的 domain，相同的 path，相同的 protocol 下。也就是说，从其它域加载的脚本通过读取 `document.cookie` 可以获取页面的 cookie。

### CSRF

samesite

参考资料：

- https://tools.ietf.org/html/rfc6265#section-8
- 《HTTP 权威指南》P278
- https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies
- https://www.nczonline.net/blog/2009/05/12/cookies-and-security/
- 《白帽子讲 Web 安全》P111
- 《Web 应用安全权威指南》P174
- 走马《cookie 之困》https://github.com/knownsec/KCon/blob/master/2015/Cookie%20%E4%B9%8B%E5%9B%B0.pdf
- https://www.google.com/intl/zh-CN/policies/technologies/types/
- https://www.owasp.org/index.php/HTTPOnly#Browsers_Supporting_HTTPOnly
- https://github.com/blog/1466-yummy-cookies-across-domains








