---
layout: post
title: "第十天，谈谈【white-space】"
description: ""
category: 
tags: []
---

`white-space` 属性用来描述元素该如何处理内部的空白。[DEMO](http://zicai.github.io/learn-css-specification-by-doing/text/white-space.html)

假设，我们有下面的HTML，代码一：

```
<div>
	a bunch of words you see
</div>	
```

我们把上面 `div` 的宽度设置为 `100px`。在正常字体下，`100px` 宽度不能容纳这么多文本。默认情况下： `white-space` 值为 `normal`，因此容器会包裹文本。如下图：

![white-space:normal](https://cdn.css-tricks.com/wp-content/uploads/2011/09/normal.png)

如果想阻止容器包裹文本，使用 `white-space:nowrap`。如下图：

![white-space:nowrap](https://cdn.css-tricks.com/wp-content/uploads/2011/09/nowrap.png)

如果你仔细看代码一，注意到其实包含两个换行，一个在文本行之前，一个在其之后。当浏览器渲染这段代码时，这些换行看起来就像被去除了一样。同时被去除的还有第一个字母之前的空白。如果想要浏览器展示这些换行和空白，可以使用 `white-space:pre;`，如下图：

![white-space:pre](https://cdn.css-tricks.com/wp-content/uploads/2011/09/pre.png)

把它叫做 `pre`，是因为它的行为就像把文本包裹在 `<pre></pre>` 标签中一样。空白会被保留与HTML代码保持一致，文本会保持在一行中，直到代码中出现换行。这在展示代码片段时很有用。

`pre` 等标签的浏览器默认的样式如下：

```
pre, xmp, plaintext, listing {
    display: block;
    font-family: monospace;
    white-space: pre;
    margin: 1em 0px;
}
```

假如，你想保留类似 `pre` 的处理空白和换行的方式，但是又需要文本换行而不是超出父容器。这就要使用 `white-space:pre-wrap`。如下图：

![white-space:pre-wrap](https://cdn.css-tricks.com/wp-content/uploads/2011/09/pre-wrap.png)

最后，`white-space:pre-line；` 会在代码中换行处换行，但是会去除空白。如下图：

![white-space:pre-line](https://cdn.css-tricks.com/wp-content/uploads/2011/09/pre-line.png)

有意思的是：最后一个换行被去除了。

在 [CSS 2.1 规范](http://www.w3.org/TR/CSS21/text.html#white-space-prop)中是这样说的：

> Lines are broken at preserved newline characters, and as necessary to fill line boxes.

下面的表格可以帮助你理解不同属性值的行为：

 &nbsp;|新行|空白与tab|包裹文本
----|----|-----|-----|
normal|合并|合并|包裹
pre|保留|保留|不包裹
nowrap|合并|合并|不包裹
pre-wrap|保留|保留|包裹
pre-line|保留|合并|包裹

在 [CSS 3](http://dev.w3.org/csswg/css3-text/#white-space) 中 the white-space property is literally going to follow that chart and map the properties to text-space-collapse and text-wrap accordingly.

### 参考链接：

- https://css-tricks.com/almanac/properties/w/whitespace/
- https://developer.mozilla.org/en-US/docs/Web/CSS/white-space