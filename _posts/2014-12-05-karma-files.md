---
layout: post
category : test
title: "karma Files"
tagline: "Supporting tagline"
tags : [karma]
---

配置项`file`数组决定了那些文件会被加载到浏览器中，哪些文件会被监测，哪些文件由karma提供

##模式匹配和 `basePath`

- 首先，所有的相对路径模式都会根据`basePath`解析
- 如果`basePath`是相对路径，那么它会根据配置文件所在目录解析
- 最终，所有模式都会通过[glob](https://github.com/isaacs/node-glob)对应到文件,所以你可以使用[minimatch](https://github.com/isaacs/minimatch)表达式，例如：`test/unit/**/*.spec.js`


##顺序

- 模式的顺序决定了文件在浏览器里被加载的顺序
- 如果多个文件匹配到同一个模式，按字母顺序加载
- 每个文件只被加载一次，如果多个模式匹配到同一个文件，由第一个匹配到的模式加载

##Included, served, watched

一个模式就是一个简单字符串或者一个有四个属性的对象：

`pattern`

- 类型： string
- 没有默认值
- 描述：需要匹配的模式，必须有值。

`watched`

- 类型：布尔值
- 默认值： `true`
- 描述：如果`autoWatch` 设为`true` ，所有`watched` 属性为`true`的文件都会被监测变化。

`included`

- 类型：布尔值
- 默认值：`true`
- 描述：浏览器是否需要通过`<script>`标签引入该文件。如果你想手动引入，例如通过`Require.js` ，就将`include`设为`false`。

`served`

- 类型：布尔值
- 默认值：`true`
- 描述：该文件是否由karma webserver 提供

##预处理器转换
详见`preprocessors`

##完整实例
下面是一个完整实例，展示了不同的选项

    files: [
    
      // 以字符串形式定义所需的测试文件
      // 等同于 {pattern: 'test/unit/*.spec.js', watched: true, served: true, included: true}
      'test/unit/*.spec.js',
    
      // 提供该文件，但是不监测该文件变化
      // 注意：如果启用了 html2js 预处理器, 以 `window.__html__['compiled/index.html']`形式引用
      {pattern: 'compiled/index.html', watched: false},
    
      // 该文件被监测变化，其它选项均为false
      {pattern: 'app/index.html', included: false, served: false}
    ],

##加载资源
默认情况下所有资源由`http://localhost:[PORT]/base/`提供，例如加载图片：

    files: [
      {pattern: 'test/images/*.jpg', watched: false, included: false, served: true}
    ],

上面的图片可以用`http://localhost:[PORT]/base/test/images/[MY IMAGE].jpg`访问到

注意URL里的`base`

或者你可以使用代理：

    proxies: {
      '/img/': 'http://localhost:8080/base/test/images/'
    },
    
现在你可以通过`http://localhost:8080/img/[MY IMAGE].jpg`获取`test/images`下的图片