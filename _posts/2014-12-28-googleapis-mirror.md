---
layout: post
category : 
title: "本地镜像googleapis 常用类库"
tagline: "Supporting tagline"
tags : [google]
---

Google为jQuery，angularjs，jQuerymobile，jQueryui等常用类库提供了CDN服务，方便广大开发者调用，同时还可以加快网站加载速度。例如：我们可以这样引用jQuery

    <script src="//ajax.googleapis.com/ajax/libs/jquery/1.9.1/jquery.min.js"></script>

但是，由于我们特殊的国情。在没翻墙的情况下访问那些使用了这些服务的网站，就会慢的像乌龟。
例如：访问 http://stackoverflow.com/

先是显示等待加载。。。

![enter image description here][1]

等待大约两分钟后，告诉你加载失败了~~~~(>_<)~~~~ 

![enter image description here][2]

由于浏览器在加载js的时候，会阻塞后面资源的加载以及内容的呈现，这就会导致呈现页面时出现明显的延迟。如果是在`<head>`标签里引用的该js，那么延迟期间浏览器窗口将是一片空白。

时间就是生命啊，完全不能忍受啊！！只是想查个资料嘛，要不要这么难！！

经过一番寻找，在github上看到了[googleapis-mirror](https://github.com/rmm5t/googleapis-mirror)。
通过它，我们可以在本地制作一份google cdn 库的镜像(不是完全镜像，是子集，包含了常用类库)。

如果你有一台可以穿墙的mac或linux，而且具备一些ruby基础，你可以跟着`googleapis-mirror`的说明，自己镜像一份。

或者直接用我做好的这份，[百度云上下载](http://pan.baidu.com/s/1dD6aZTJ)。

当然不管用哪种方法，你本地都要有服务器环境，我用的是apache

1. 下载完成，解压到本地 apache 的 www 目录
2. 修改hosts文件，将`ajax.googleapis.com` 指向到 `127.0.0.1`
3. 修改apache虚拟主机配置，并重启。

        <VirtualHost *:80>
            ServerName ajax.googleapis.com
            DocumentRoot "/path/to/googleapis-mirror"
               <Directory "/path/to/googleapis-mirror">
                 Options Indexes
                 Order allow,deny
                 Allow from all
               </Directory>
        </VirtualHost>

       `/path/to/googleapis-mirror`要改成你对应的目录。

再次访问stackoverflow

![enter image description here][3]

速度杠杠的。


类似的，有许多网站用到了google字体库(fonts.googleapis.com)中的字体，同样也会导致网页加载很慢。

我的解决方法简单粗暴，修改hosts文件，把`fonts.googleapis.com` 指到 `127.0.0.1`。直接使字体加载失败，损失一些美观度，节约等待时间。

  [1]: http://htmljs.b0.upaiyun.com/uploads/1419752375122-QQ%E5%9B%BE%E7%89%8720141228153919.jpg
  [2]: http://htmljs.b0.upaiyun.com/uploads/1419752506204-QQ%E5%9B%BE%E7%89%8720141228153924.png
  [3]: http://htmljs.b0.upaiyun.com/uploads/1419753926926-QQ%E5%9B%BE%E7%89%8720141228160509.png