---
layout: post
category : 
title: "走向 Retina Web【译】"
tagline: "Supporting tagline"
tags : [retina]
---

在深入细节之前，有必要先明确一些关键性概念。

##Device Pixels(设备像素)

![物理像素][1]

一个设备像素（或者称为物理像素）是显示器上最小的物理显示单元。在操作系统的调度下，每一个设备像素都有自己的颜色值和亮度值。

###Screen density(屏幕密度)

屏幕密度指的是单位面积里物理像素的数量，通常以PPI(pixels per inch)为单位。苹果公司为它的双倍屏幕密度的显示器（double-density displays）创造了一个新词“Retina”，声称在正常的观看距离下，人眼无法在Retina显示器上分辨出单独的像素。

##CSS Pixels

![CSS Pixels][2]

CSS pixel是浏览器使用的抽象单位，用来精确的、统一的绘制网页内容。通常，CSS pixels被称为与设备无关的像素（DIPs,device-independent pixels）。在标准密度显示器（standard-density displays）上，1 CSS pixel对应一个物理像素。

    <div height="200" width="300"></div>

在标准密度显示器上，上面的`div`会占据200 * 300 个物理像素。而在Retina显示器上，为了保持相同的物理大小，上面的`div`需要用400 * 600 个物理像素来渲染，如下图：

![On a Retina display, four times as many device pixels are on the same physical surface][3]

物理像素与CSS pixel 的比率可以通过媒体查询的`device-pixel-ratio`来检测（[`device-pixel-ratio`兼容性](http://caniuse.com/#search=device-pixel-ratio)）。也可以通过javascript的`window.devicePixelRatio`来获取该比率。

##Bitmap Pixels（位图像素）

![位图像素][4]

一个位图像素是栅格图像（也就是位图，png、jpg、gif等等）最小的数据单元。每一个位图像素都包含着该如何显示自己的信息，例如显示位置、颜色值等。一些图片格式还包含额外的数据，例如透明度。

除了自身的分辨率外，图片在网页上还有一个抽象的尺寸，通过CSS pixels来定义。浏览器在渲染的过程中，会根据图片的CSS高度和宽度来压缩或是拉伸图片。

当一个位图以原尺寸展示在标准密度显示器上时，一位图像素对应一个物理像素，就是无失真显示。而在Retina显示器上，为了保证同样的物理尺寸，需要用四倍的像素来展示，但由于单个位图像素已经无法再进一步分割，只能就近取色，导致图片变虚。

![Each bitmap pixel gets multiplied by four to fill the same physical surface on a Retina display.][5]

##解决方案

下面的每一种方案都是权衡性能、实现难度、跨浏览器兼容性等之后的结果。你需要根据实际情况进行选择。

##方案一：通过HTML 和 CSS 控制大小

最直接的方式就是通过CSS或者HTML将图片尺寸减半。例如，要提供一个200 * 300像素的图片（这里指的是CSS pixels）,你需要上传一张尺寸为400 * 600像素的图片到服务器，然后通过CSS属性或者HTML属性将其缩小50%。在标准密度显示器上，显示结果就是一张只有原图像素总数四分之一的图片，这个过程通常被称为downsampling。

![A CSS-sized image gets its dimensions halved during the rendering process.][6]

而在Retina显示器上，将会用四倍的物理像素数来渲染同一张图片，这样每一个物理像素就对应一个位图像素，从而图片的每一个像素都得到展现。

![CSS-sized images regain their full-detail glory on Retina displays.][7]

通过下面几种技术可以实现上面的解决方案：

###通过HTML

最简单的方式就是使用`img`标签的`width`和`height`属性：

    <img src="example@2x.png" width="200" height="300" />

需要说明：虽然指定图片高度是可选的，但是声明高度的好处是，浏览器会在图片加载之前预留出相应的位置，这就避免了图片加载完成后页面布局发生变化。


###通过javascript

利用jQuery，可以这样写：

    $(window).load(function() {
      var images = $('img');
        images.each(function(i) {
          $(this).width($(this).width() / 2);
        });
    });


###通过CSS（SCSS）

如果你想把有关样式的代码都放到css里的话，那么最常见的方法就是使用HTML元素的背景替换`img`标签，然后指定`background-size`属性。你可以明确指定背景图片的宽度和高度，或者是在指定了HTML元素的宽高的前提下使用`background-size: contain`。需要注意的是IE7、8不支持`background-size`

    .image {
      background-image: url(example@2x.png);
      background-size: 200px 300px;
      /* 或者是用 background-size: contain; */
      height: 300px;
      width: 200px;
    }

还可以通过`:before`或者`:after`伪元素来实现：

    .image-container:before {
      background-image: url(example@2x.png);
      background-size: 200px 300px;
      content:'';
      display: block;
      height: 300px;
      width: 200px;
    }

该技术对CSS sprite同样适用，因为`background-position`指定的值是相对于CSS 大小的（在这里就是 200px * 300px）:

    .icon {
      background-image: url(example@2x.png);
      background-size: 200px 300px;
      height: 25px;
      width: 25px;
      &.trash {
        background-position: 25px 0;
      }
      &.edit {
        background-position: 25px 25px;
      }
    }

方案一的优势：

- 容易实现
- 跨浏览器兼容

方案一的不足：

- 非Retina设备需要下载Retina资源
- 在标准密度屏幕上Downsampled的图片可能会丢失一些锐利度
- IE7、8不支持`background-size`

##方案二：查询像素密度
应对Retina最流行的方式应该就是通过查询像素密度，然后针对不同密度提供不同的资源。该方案可以用CSS或javascript来实现。

###通过CSS媒体查询
大多数浏览器都以私有前缀实现了`device-pixel-ratio`，以及它的俩兄弟`min-device-pixel-ratio` 和 `max-device-pixel-ratio`。媒体查询再结合`background-image`：

    .icon {
      background-image: url(example.png);
      background-size: 200px 300px;
      height: 300px;
      width: 200px;
    }
    
    @media only screen and (-Webkit-min-device-pixel-ratio: 1.5),
    only screen and (-moz-min-device-pixel-ratio: 1.5),
    only screen and (-o-min-device-pixel-ratio: 3/2),
    only screen and (min-device-pixel-ratio: 1.5) {
      .icon {
        background-image: url(example@2x.png);
      }
    }

这里使用的比率是1.5而不是2，这样该媒体查询还可以覆盖一些非苹果设备。

CSS媒体查询优势：

- 设备只需下载相匹配的资源
- 跨浏览器兼容
- 像素级精确控制

CSS媒体查询不足：

- 代码冗长，尤其是大网站
- 把应该由`img`标签展示的图片转化为背景图片属于语义错误。

###通过javascript

像素密度可以通过javascript的`window.devicePixelRatio`来查询(注意：不是所有浏览器都支持`devicePixelRatio`)。一旦检测到高密度显示器，你就可以用高质量图片替换普通图片：

    $(document).ready(function(){
      if (window.devicePixelRatio > 1) {
        var lowresImages = $('img');
        images.each(function(i) {
          var lowres = $(this).attr('src');
          var highres = lowres.replace(".", "@2x.");
          $(this).attr('src', highres);
        });
      }
    });

[Retina.js](http://retinajs.com/)是一个js插件，原理类似，但是有一些额外功能，例如：忽略外部图片等等。

通过javascript查询的优势：

- 容易实现
- 非Retina设备无需下载大尺寸图片
- 像素级精确控制

通过javascript查询的不足：

- Retina设备不得不下载标准图片和高质量图片
- 图片替换的过程会被用户看到
- 有一些浏览器不支持（例如IE和火狐）


##方案三：SVG(Scalable Vector Graphics)
由于位图本身固有的性质，不可能无限制的缩放。而恰恰这是矢量图的优势所在。

浏览器对svg的[兼容程度](http://caniuse.com/#search=svg)

使用svg资源最直接的方式就是通过`img`标签或者CSS `background-image` 或者`content:url()`。

    <img src="example.svg" width="200" height="300" />

或是通过CSS:

    /* 使用 background-image */
    .image {
      background-image: url(example.svg);
      background-size: 200px 300px;
      height: 200px;
      width: 300px;
    }
    
    /* 使用 content:url() */
    .image-container:before {
      content: url(example.svg);
      /* width 和 height 对 content:url() 不起作用 */
    }

使用svg不仅可以节约宝贵的带宽资源，还可以使你的图片资源更容易维护。

如果你需要支持IE7、8或是Android 2.x，那么你还需要一个fallback 方案：用对应的PNG资源替换SVG图片。通过[Modernizr](http://modernizr.com/docs/#features-misc) 来做，很简单：

    .image {
      background-image: url(example.png);
      background-size: 200px 300px;
    }
    .svg {
      .image {
        background-image: url(example.svg);
      }
    }

也可以通过html实现类似的fallback方案：给`img`标签添加自定义`data`属性：

    <img src="example.svg" data-png-fallback="example.png" />

然后，剩下的就交给jQuery和Modernizr

    $(document).ready(function(){
      if(!Modernizr.svg) {
        var images = $('img[data-png-fallback]');
        images.each(function(i) {
          $(this).attr('src', $(this).data('png-fallback'));
        });
      }
    });

通过html实现的fallback方案缺点就是：不支持svg的浏览器也会下载svg资源。

方案三的优势：

- 所有设备用同一套资源
- 容易维护
- 应对未来的变化：可以无限缩放

方案三的不足：

- 由于anti-aliasing，所以不能精确到像素
- 不适合复杂的的图像，因为图片体积会非常大
- IE7、8以及早期android没有原生支持svg

##方案四：Icon Fonts

![Icon Fonts][8]

Twitter的 bootstrap使Icon Fonts 更加的流行，该技术是通过`@font-face`引入基于icon的字体来代替位图icon，这样icon就不再受分辨率影响。用纯色icon代替字母的web Fonts，可以用CSS来调整样式，就像网页里其它文本一样。

你可以通过[Fontello](http://fontello.com/), [Font Builder](https://github.com/fontello/font-builder) 和[Inkscape](http://www.webdesignerdepot.com/2012/01/how-to-make-your-own-icon-Webfont/)来制作自己的font。

网页中使用icon fonts 最常见的方式是给特定的HTML元素(通常是`<span>`或`<i>`)赋予`.icon` 或`.glyph` class,然后使用icon对应的字符作为内容：

    <span class="icon">a</span>

通过`@font-face`引入了自定义font后，用下面的方式使用：

    .icon {
      font-family: 'My Icon Font';
    }

另一种方式是使用`:before`伪元素和`content`属性，每个icon对应一个class：

    <span class="glyph-heart"></span>

CSS:

    [class^="glyph-"]:before {
      font-family: 'My Icon Font';
    }
    .glyph-heart:before {
      content: 'h';
    }

方案四的优势：

- 应对未来的变化：可以无限制缩放
- 跨浏览器
- 比图片资源更灵活：可以用在placeholder文本里以及其他form元素中

方案四的不足：

- 由于anti-aliasing，所以不能精确到像素
- 比较难维护：修改一个icon就需要重新生成整个font
- 依赖语义错误的标签（除非是用`:before` 或`:after`伪元素）


##Favicons
Favicons也获得了它应有的关注度，越来越多的应用于浏览器之外，用作网站或应用的图标。
要想适配Retina，需要导出两版.ico文件，16 * 16像素 和 32 * 32像素。

##一睹未来
除了上面提到的几种技术，还有一些技术值得关注：

- `-Webkit-image-set`用来提供多版本背景图
 
    例如：

        .image {
          background-image: -Webkit-image-set(url(example.png) 1x, url(example@2x.png) 2x);
          background-size: 200px 300px;
        }

-  [Picturefill](https://github.com/scottjehl/picturefill)是一个html+js解决方案，大量使用`data`属性和媒体查询，来给不同的媒体上下文提供不同的图片。

    例如：

    	<div data-picture>
         	<div data-src="example.png"></div>
         	<div data-src="example@2x.png" data-media="(min-device-pixel-ratio: 1.5)"></div>
      	<!-- Fallback content for non-JS browsers -->
      	<noscript>
        	<img src="example.png" >
      	</noscript>
    	</div>
    


原文地址：[http://www.smashingmagazine.com/2012/08/20/towards-retina-web/](http://www.smashingmagazine.com/2012/08/20/towards-retina-web/)

ps:虽然原文发表于2012/08/20，但对于不知该如何应对Retina的前端来说，信息量还是非常大的。

  [1]: http://media.mediatemple.netdna-cdn.com/wp-content/uploads/2012/07/device-pixels.png
  [2]: http://media.mediatemple.netdna-cdn.com/wp-content/uploads/2012/07/css-pixels.png
  [3]: http://media.mediatemple.netdna-cdn.com/wp-content/uploads/2012/07/css-device-pixels.png
  [4]: http://media.mediatemple.netdna-cdn.com/wp-content/uploads/2012/07/bitmap-pixels.png
  [5]: http://media.mediatemple.netdna-cdn.com/wp-content/uploads/2012/07/css-device-bitmap-pixels.png
  [6]: http://media.mediatemple.netdna-cdn.com/wp-content/uploads/2012/07/downsampling.png
  [7]: http://media.mediatemple.netdna-cdn.com/wp-content/uploads/2012/07/html-sizing.png
  [8]: http://media.mediatemple.netdna-cdn.com/wp-content/uploads/2012/07/icon-fonts.png