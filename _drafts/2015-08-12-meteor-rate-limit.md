---
layout: post
category : lessons
title: "Rate limiting in Meteor core"
tagline: "Supporting tagline"
tags : [meteor]
---

在这篇文章中，我们将谈论大多数Web应用程序一个共同的需求--速率限制。速率限制是阻止用户发送垃圾内容，用光服务器资源的第一道防线。

## 目前 Web 框架速率限制的调查

目前，大多数流行​​的 Web 框架都不提供开箱即用的针对拒绝服务攻击（DoS）或字典密码攻击的安全机制。谷歌搜索 Web框架的速率限制，搜索结果显示有很多不同的速率限制选项：例如，[Flask that involves using Redis](http://flask.pocoo.org/snippets/70/) 和 [Rails的一个社区包](https://github.com/bendiken/rack-throttle)。然而，框架默认都不包含这些解决方案。

速率限制，也可以是 Web 服务器或代理的工作，在这个层面上也有许多解决方案，但是针对 WebSockets 的速率限制还很不成熟。

## Meteor 中实现速率限制

在 Meteor 社区的帮助下，我们确定了速率限制的重要性，并因此在 Meteor 1.2 中默认包含了速率限制解决方案。Meteor 使用的 websocket 协议是 DDP，用来从服务端获取数据以及当数据发生变化时接收实时更新。

很多 web 框架很容易受到密码碰撞脚本的攻击。我们的解决方法是在 `accounts-base` 包中限制了尝试密码的速度。这个限制将保护 Meteor APP，避免被恶意重置密码，创建新用户或是用户名/密码爆破攻击。

除了对用户帐户的内置支持，我们还为 DDP 提供了一个更通用的速率限制器，这样开发者就可以限制方法调用和数据订阅的速率。速率限制器在连接层面进行拦截，允许我们判断是否已经达到了限制，返回一个错误提示，而不会使用更多的服务器资源。

## 如何轻松地添加速率限制到应用程序

下面是一个代码示例，在 Meteor 1.2 中给 DDPRateLimiter 添加规则：

```
// Define a rule that matches add comment attempts by non-admin users
var addCommentRule = {
    userId: function (userId) {
        return Meteor.users.findOne(userId).type !== 'Admin';
    },
    type: 'method',
    method: 'addComment'
}

// Add the rule, allowing up to 5 messages every 1000 milliseconds.
DDPRateLimiter.addRule(addCommentRule, 5, 1000);
```

解释下上面的代码：

- 规则被定义为一组键值对，其中键是`userId`，`clientAddress`，`type`，`name`，和`ConnectionId`中的一个或多个。值可以为空，原始值或是函数。当你想限制一些用户，使用函数可以在运行时基于数据库或其他数据确定是否触发限制。在上面的例子中，检查数据库以避免限制管理员用户。
- 当把规则添加到DDPRateLimiter，还需指定允许的次数以及重置速率限制的时间间隔。

原文地址：[http://info.meteor.com/blog/rate-limiting-in-meteor-core](http://info.meteor.com/blog/rate-limiting-in-meteor-core)
