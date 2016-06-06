---
layout: post
category : lessons
title: "Node API 学习--DNS"
tagline: "Supporting tagline"
tags : [node,dns]
---
dns 模块包含两类不同的函数

- 使用底层操作系统提供的功能来进行域名解析。dns.lookup()
- 直接通过 DNS server 来进行

```
dns.lookup(hostname[,options],callback)
```


```
dns.lookupService(address, port, callback)
```

```
dns.resolve(hostname[, rrtype], callback)
```
使用 DNS 协议，根据 rrtype 指定的记录类型将主机名（hostname）解析为数组。

rrtype 值可以是：

- 'A' - IPV4 地址，默认值
- 'AAAA' - IPV6 地址
- 'MX' - mail exchange 记录
- 'TXT' - text 记录
- 'SRV' - SRV 记录
- 'PTR' - PTR 记录
- 'NS' - name server 记录
- 'CNAME' - canonical name 记录
- 'SOA' - start of authority 记录
- 'NAPTR' - name authority pointer 记录

callback 函数参数为 (err, addresses)。解析成功时，addresses 是一个数组。


- `dns.resolve4(hostname, callback)`
- `dns.resolve6(hostname, callback)`
- `dns.resolveCname(hostname, callback)`
- `dns.resolveMx(hostname, callback)`
- `dns.resolveNaptr(hostname, callback)`
- `dns.resolveNs(hostname, callback)`
- `dns.resolveSoa(hostname, callback)`
- `dns.resolveSrv(hostname, callback)`
- `dns.resolvePtr(hostname, callback)`
- `dns.resolveTxt(hostname, callback)`



```
dns.reverse(ip, callback)
```
执行反向 DNS 查询，将 IPv4 或 IPv6 地址解析为域名数组。


```
dns.setServer(servers)
```

设置用于解析的服务器的 IP。参数 servers 是一个 IPv4 或 IPv6 的数组


```
dns.getServers()
```


参考地址：

- https://nodejs.org/api/dns.html