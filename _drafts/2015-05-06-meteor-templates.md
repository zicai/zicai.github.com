---
layout: post
category : lessons
title: "meteor 模板"
tagline: "Supporting tagline"
tags : [meteor]
---


当你在HTML文件中写了一个模板`<template name="foo"> ... </template>`后，Meteor会生成一个模板对象，叫做`Template.foo`。

相同的模板可以在一个页面中出现多次，每一个叫做一个实例。模板实例的生命周期为：创建，插入到页面，从页面中移除，销毁。Meteor替你管理这些阶段，包括当模板实例从页面中移除或被替换时，确定何时应该进行清理。你可以给模板实例关联数据，当模板插入到页面后，你可以获取DOM节点。

```
Template.myTemplate.events(eventMap)		client
给模板指定事件处理器

参数
	eventMap	EventMap
	
```
可以多次调用。


```
Template.myTemplate.helpers(helpers)		client
给模板声明helper

参数
	helpers Object
	
```

例如：

```
Template.myTemplate.helpers({
  foo: function () {
    return Session.get("foo");
  }
});
```

现在就可以在模板`<template name="myTemplate">`中用`{{foo}}`调用helper

helper可以接受位置参数和关键字参数：

```
Template.myTemplate.helpers({
  displayName: function (firstName, lastName, keyword) {
    var prefix = keyword.hash.title ? keyword.hash.title + " " : "";
    return prefix + firstName + " " + lastName;
  }
});
```

可以用下面的方式调用上面的helper

	{{displayName "John" "Doe" title="President"}}
	
有关helper 参数更多内容参见Spacebars readme

在底层，每一个helper都创建了一个新的Tracker.autorun。当它的reactive依赖变化时，helper会重新运行。helper依赖于它的数据上下文，传入的参数和在执行过程中其它reactive数据源

```
Template.myTemplate.onRendered		client
注册一个函数，当一个模板实例插入到DOM中时被调用

参数
	callback		Function
	
```

当模板实例渲染为DOM节点，第一次插入到文档中的时候调用回调函数。

在回调函数中，this是一个模板实例对象






**Event Map**
event map 是一个对象，属性指定了要处理的事件，属性值是事件处理器。属性可以是下面几种形式：

- eventtype
	匹配特定的事件类型，例如'click'
- eventtype selector
	匹配指定元素上的事件
- event,event2
	用相同的函数处理多个事件，用逗号分隔
	
事件处理函数接受两个参数：event 包含事件信息的对象，template 事件处理函数所处的模板实例。事件处理器内部，通过this获取上下文数据，上下文取决于处理事件的元素的上下文。在模板中，一个元素的上下文就是它出现位置的上下文。

例如：

```
{
  // Fires when any element is clicked
  'click': function (event) { ... },

  // Fires when any element with the 'accept' class is clicked
  'click .accept': function (event) { ... },

  // Fires when 'accept' is clicked or focused, or a key is pressed
  'click .accept, focus .accept, keypress': function (event) { ... }
}
```

大多数事件从源元素开始沿着文档树冒泡。事件的源元素通过target属性获取，而匹配选择器，正在处理事件的元素通过currentTarget获取。

```
{
  'click p': function (event) {
    var paragraph = event.currentTarget; // always a P
    var clickedElement = event.target; // could be the P or a child element
  }
}
```

如果选择器匹配了冒泡过程中的多个元素，那么事件处理器会被调用多次，例如'click
div' 或 'click *'。如果没有给定selector，事件处理函数只会在源头元素上调用一次。

传给事件处理器的事件对象包含下面的属性和方法：

	type	String
	事件类型，例如 click,blur,keypress
	target		DOM Element
	事件的起源元素
	currentTarget		DOM Element
	当前正在处理事件的元素。由于事件会冒泡，它可能是target也可能是target的祖先，它的值随着冒泡而变化。
	which		Number
	对于鼠标事件，是鼠标按钮的数字(1=left,2=middle,3=right)。对于键盘事件，是字符或key code
	stopPropagation()
	阻止事件冒泡到其他元素。当前元素的事件处理器仍然会被调用。
	stopImmediatePropagation()
	停止所有的事件处理器运行，包括在当前event map中的，冒泡触发的，和在其他event map中的事件处理器。
	preventDefault()
	阻止浏览器的默认行为，例如跟踪超链接，提交表单。其它的事件处理器仍会被调用，但是不能reverse the effect
	isPropagationStopped()
	返回是否在这个事件上调用了stopPropagation()
	isImmediatePropagationStopped()
	返回是否在这个事件上调用stopImmediatePropagation()
	isDefaultPrevented()
	返回是否在这个事件上调用了preventDefault()
	
在一个事件处理器中返回false,等同于在事件上调用stopImmediatePropagation 和preventDefault。

事件类型包括：

	click
	鼠标点击任何元素，包括链接，按钮，表单，或是div。一些用键盘激活元素的方法也会触发click。
	dblclick
	foucs,blur
	事件不冒泡
	change
	单选和多选按钮改变状态。对于文本框，用blur或是key event 来响应change
	mouseenter,mouseleave
	鼠标指针移入移出元素的边界。事件不冒泡
	mousedown,mouseup
	keydown,keypress,keyup
	捕获用户在文本框的输入，keypress非常有用。keydown 和keyup 用于方向键和modifier key.
	
其它的DOM事件也支持，但是对于上面的事件，Meteor确保了浏览器行为的一致性。 










