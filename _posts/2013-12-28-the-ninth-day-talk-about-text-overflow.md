---
layout: post
title: "第九天，谈谈【text-overflow】"
description: ""
category: 
tags: []
---

text-overflow 的作用:
-----------------

`text-overflow` 用来声明：当内容超出容器宽度时做何处理。

text-overflow 语法:
------------------

    text-overflow: [ clip | ellipsis | <string> ]

简单的说明：`text-overflow`的默认值为`clip`，即当内容超出容器时，会裁切掉超出的文本；当`text-overflow`值为`ellipsis`时，会用省略号替代超出的文本；也可以用特定的字符串来替代超出的文本（目前仅 firefox 支持）。

text-overflow 起作用的前提:
--------------------

 1. 容器要有一定的宽度，没有宽度就谈不上内容超出容器。例如：对一个普通的inline元素应用`text-overflow`不会有效果。
 2. 容器的`overflow`属性值不能为`visible`。因为`overflow:visible`会使内容溢出容器。同时为了避免出现滚动条，通常不使用`overflow: auto` 、 `overflow: scroll`而采用`overflow: hidden`
 3. 容器必须能强制内容不换行，例如使用：`white-space: nowrap`

text-overflow 示例:
-------------------
<iframe width="100%" height="300" src="http://jsfiddle.net/zicai/PFx7g/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>




**省略号样式** 总结如下：

    .ellipsis{
        overflow: hidden;
        white-space: nowrap;
        text-overflow: ellipsis;
    }


参考资料：

 - [http://quirksmode.org/css/user-interface/textoverflow.html](http://quirksmode.org/css/user-interface/textoverflow.html)
 - [https://developer.mozilla.org/en-US/docs/Web/CSS/text-overflow](https://developer.mozilla.org/en-US/docs/Web/CSS/text-overflow)



  [1]: http://jsfiddle.net/zicai/PFx7g/embedded/