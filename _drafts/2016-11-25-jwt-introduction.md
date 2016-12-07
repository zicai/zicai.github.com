---
layout: post
category : learn
title: "JWT 介绍"
tagline: "Supporting tagline"
tags : [jwt]
---


JSON Web Token (JWT) 是一个开放标准（RFC 7915），它定义了一种简洁、自包含的方式，可以用来安全的交换 JSON 形式的信息。因为有数字签名，所以信息是可以验证和信任的。JWT 可以使用 HMAC 算法或是 RSA 算法进行签名。

- 简洁：体积小，可以通过 URL 或是 POST 参数或是 HTTP 头 发送。体积小也意味着速度快
- 自包含：payload 包含关于用户所有需要的信息，避免多次查询数据库。

## 什么时候可以用 JWT

- 认证（Authentication）：这是使用 JWT 最常见的场景。用户登录之后，后续的每个请求都带有 JWT，允许用户访问那些 token 允许的路由、服务和资源。现在 SSO 大量使用了 JWT，因为它开销少，并且可以很容易的跨域。
- 信息交换：JWT 是一种安全的传递信息的好方法，因为它有签名。

## JWT 的结构

JWT 由三部分组成，点号分割：

```
Header.Payload.Signature
```

### Header
头部通常包含两部分：

- token 的类型，也就是 JWT
- 用到的哈希算法，例如：HMAC SHA256 或 RSA.

例如：

```
{
  "alg": "HS256",
  "typ": "JWT"
}
```

然后把头部 JSON 对象进行 Base64Url 编码作为JWT 的第一部分。

### Payload

payload 包含 claims。claims 就是有关实体（通常是用户）及其额外的元数据的声明。有三种类型的 claims：

- Reserved claims：规范预定义的 claims，虽然不是强制的，但是为了互操作性还是推荐使用。包括：
	- iss (issuer)：JWT 的签发者
	- exp (expiration time)：过期时间，Unix 时间戳
	- sub (subject)：JWT 所面向的用户
	- aud (audience)：接收该 JWT 的一方
- Public claims
- Private claims

例如：

```
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

同样的，payload 也要 Base64Url 编码，作为 JWT 的第二部分。

### Signature

构建 JWT 的第三部分签名需要用到：编码后的 header，编码后的 payload，一个 secrete 以及 header 中指定的算法。

例如：假设你要使用 HMAC SHA256 算法，签名的生成方式如下：

```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

签名用来验证发送者的身份，同时保证中途未被篡改。

### 组合到一起
经过上面三步，得到了三个 Base64 字符串，用点号连接起来就得到了 JWT。对比基于 XML 的规范，例如 SAML，JWT 更简洁。

[http://jwt.io/](http://jwt.io/) 提供了在线解码、验证、生成 JWT 的功能。

## 如何使用 JWT

认证的时候，当用户成功登录，服务器返回一个 JWT，客户端保存它（通常是 local storage，也可以使用 cookie），区别传统的会话机制：在服务端创建一个 session，返回一个 cookie。

无论何时，用户想要获取受保护的路由或资源，用户代理都要发送 JWT，通常是在 Authorization 头，使用 Bearer schema。请求头内容类似下面：

```
Authorization: Bearer <token>
```

JWT 是一种无状态认证机制，因为用户状态不用保存在服务端内存。服务端受保护的路由会检查 Authorization 头里 JWT 是否合法，只有合法时，才允许访问受保护的资源。因为 JWT 是自包含的，所需信息都有，省去了多次查询数据库。

This allows you to fully rely on data APIs that are stateless and even make requests to downstream services. 使用哪个域名来提供 API 无关紧要，因为不使用 cookie，所以 CORS 不是问题。




## JWT 的优势在哪？

JWT 相比于 Simple Web Tokens (SWT) 和 Security Assertion Markup Language Tokens (SAML) 的优势在于：

- JSON 比 XML 更简洁，编码后的体积更小，所以 JWT 比 SAML 更简洁。使得 JWT 适合在 HTML 和 HTTP 环境中使用。
- 安全方面：SWT can only be symmetrically signed by a shared secret using the HMAC algorithm. However, JWT and SAML tokens can use a public/private key pair in the form of a X.509 certificate for signing. Signing XML with XML Digital Signature without introducing obscure security holes is very difficult when compared to the simplicity of signing JSON.
- 编程语言中，JSON 更容易映射为对象

注意：

这个协议只用来认证，不负责加密。只要拿到token，就拿到里面的信息，所以不要将敏感信息写入 payload。

参考资料：

- https://tools.ietf.org/html/rfc7519
- https://jwt.io/introduction/
- https://github.com/auth0/node-jsonwebtoken
- http://blog.leapoahead.com/2015/09/06/understanding-jwt/








