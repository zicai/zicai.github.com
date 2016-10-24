---
layout: post
category : lessons
title: "box-shadow 属性【译】"
tagline: "Supporting tagline"
tags : [css,box-shadow]
---

PS：非直译。有翻译不当的地方，请指出。

CSS3 引入的 `box-shadow` 属性用来创建阴影效果。阴影效果给我们的二维平面增加了深度的感觉。

## 语法

`box-shadow` 属性值由五部分组成：

```
box-shadow: offset-x offset-y blur spread color position;
```

一定要注意顺序。

### offset-x

offset-x 用来声明阴影的水平偏移，即阴影在 X 轴上的位置。当值为正数，阴影位于元素的右侧，若值为负数，阴影位于元素的左侧。

```
.left { box-shadow: 20px 0px 10px 0px rgba(0,0,0,0.5) }

.right { box-shadow: -20px 0px 10px 0px rgba(0,0,0,0.5) }
```

![Demo of offset-x positive and negative values](https://bitsofco.de/content/images/2016/05/offset-x.png)

### offset-y

offset-y 用来声明阴影的垂直偏移，即阴影在 Y 轴上的位置。当值为正数，阴影位于元素的下面，若值为负数，阴影位于元素的上面。

```
.left { box-shadow: 0px 20px 10px 0px rgba(0,0,0,0.5) }

.right { box-shadow: 0px -20px 10px 0px rgba(0,0,0,0.5) }
```

![Demo of offset-y positive and negative values](https://bitsofco.de/content/images/2016/05/y.png)
### blur

blur 表示阴影的模糊半径。效果与设计软件中使用的高斯模糊滤镜一样。值为 0 意味着阴影完全不模糊。blur 值越大，边角越不锋利，阴影越朦胧。不允许负值，负值等同于 0。

```
.left { box-shadow: 0px 0px 0px 0px rgba(0,0,0,0.5) }

.middle { box-shadow: 0px 0px 20px 0px rgba(0,0,0,0.5) }

.right { box-shadow: 0px 0px 50px 0px rgba(0,0,0,0.5) }
```

![Demo of blur value at 0px, 20px and 50px](https://bitsofco.de/content/images/2016/05/blur.png)
### spread

spread 表示阴影的大小。也可以把这个值看做阴影与元素之间的距离。当值为正数，阴影会向四周扩展。若值为负数，阴影会收缩，小于元素尺寸。默认值 0 会保持阴影和元素尺寸一致。

```
.left { box-shadow: 0px 0px 20px 0px rgba(0,0,0,0.5) }

.middle { box-shadow: 0px 0px 20px 20px rgba(0,0,0,0.5) }

.right { box-shadow: 0px 50px 20px -20px rgba(0,0,0,0.5) }
```

![Demo of spread value at 0px, 20px and -20px](https://bitsofco.de/content/images/2016/05/spread.png)

### color

color 表示阴影的颜色。可以是任何颜色单位。参见：[working with colour in css](https://bitsofco.de/working-with-colour-in-css/)

```
.left { box-shadow: 0px 0px 20px 10px #67b3dd }

.right { box-shadow: 0px 0px 20px 10px rgba(0,0,0,0.5) }
```

![Demo of color value ](https://bitsofco.de/content/images/2016/05/Screen-Shot-2016-05-23-at-10-20-23.png)

### position

position 表示阴影的位置，可选项。默认为外部阴影。可以通过使用 inset 关键字来制作内部阴影。

```
.left { box-shadow: 20px 20px 10px 0px rgba(0,0,0,0.5) inset }

.right { box-shadow: 20px 20px 10px 0px rgba(0,0,0,0.5) }
```

![Demo of position values as inset of default](https://bitsofco.de/content/images/2016/05/position.png)

### 多重阴影

一个元素的 `box-shadow` 属性可以接受多个阴影声明，组成一个逗号分隔的列表。

```
.foo { 
    box-shadow: 20px 20px 10px 0px rgba(0,0,0,0.5) inset, /* inner shadow */
                20px 20px 10px 0px rgba(0,0,0,0.5); /* outer shadow */
}    
```

![Demo of multiple box shadows](https://bitsofco.de/content/images/2016/05/multiple.png)

### 圆角阴影

`box-shadow` 属性的 border-radius 由同一个元素的 `border-radius` 属性来控制。

```
.foo { 
    box-shadow: 20px 20px 10px 0px rgba(0,0,0,0.5);
    border-raduis: 50%;
}
```

![Demo of rounded shadow](https://bitsofco.de/content/images/2016/05/radius.png)

## 组合起来

把上面各部分组合起来，我们可以用 `box-shadow` 来创建一些惊人的效果。

### Non-Layout-Blocking 边框的另一种选择

用 `box-shadow` 模拟边框，不会影响盒模型以及页面其余部分的布局。利用多重阴影，可以使元素有不同颜色的边框。

```
.simple {
    box-shadow: 0px 0px 0px 40px indianred;
}

.multiple {
    box-shadow: 20px 20px 0px 20px lightcoral,
                -20px -20px 0px 20px mediumvioletred,
                0px 0px 0px 40px rgb(200,200,200);
}
```

![Border using the box-shadow property](https://bitsofco.de/content/images/2016/05/Screen-Shot-2016-05-21-at-13-07-08.png)

### pop-up 效果

通过对 `box-shadow` (& `transform`) 属性进行变换，可以模拟元素靠近和远离用户的效果。

```
.popup {
    transform: scale(1);
    box-shadow: 0px 0px 10px 5px rgba(0, 0, 0, 0.3);
    transition: box-shadow 0.5s, transform 0.5s;
}
.popup:hover {
    transform: scale(1.3);
    box-shadow: 0px 0px 50px 10px rgba(0, 0, 0, 0.3);
    transition: box-shadow 0.5s, transform 0.5s;
}
```

![Pop-up effect using the box-shadow property](https://bitsofco.de/content/images/2016/05/popup.gif)

### floating 效果

给 `:after` 伪元素添加 `box-shadow`。在元素下方创建阴影，来模拟升起和下降。

```
.floating {
    position: relative;
    
    transform: translateY(0);
    transition: transform 1s;
}
.floating:after {
    content: "";
    display: block;
    position: absolute;
    bottom: -30px;
    left: 50%;
    height: 8px;
    width: 100%;
    box-shadow: 0px 0px 15px 0px rgba(0, 0, 0, 0.4);
    border-radius: 50%;
    background-color: rgba(0, 0, 0, 0.2);

    transform: translate(-50%, 0);
    transition: transform 1s;
}

/* Hover States */
.floating:hover {
    transform: translateY(-40px);
    transition: transform 1s;
}
.floating:hover:after {
    transform: translate(-50%, 40px) scale(0.75);
    transition: transform 1s;
}
```

![Floating effect using the box-shadow property](https://bitsofco.de/content/images/2016/05/floating.gif)

`box-shadow` 不单单是一个创建阴影的工具，使用它还可以创建很多其它复杂的效果。例如：[纸张效果](http://codepen.io/haibnu/pen/FxGsI)。

原文地址：

* [https://bitsofco.de/the-box-shadow-property/](https://bitsofco.de/the-box-shadow-property/)
