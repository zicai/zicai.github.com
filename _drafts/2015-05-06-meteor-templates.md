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

在回调函数中，this是一个模板实例对象 that is unique to this occurrence of the template and persists across re-renderings。使用onCreated 和onDestroyed 回调函数来对该对象进行初始化和清理。

```
Template.myTemplate.onCreated		client
注册一个函数，当模板实例创建时调用。

参数
	callback		Function
```

在回调中，this是模板对象实例。你给这个对象设置的属性，在onRendered 和onDestroyed 回调中以及事件处理器中均可见。

These callbacks fire once and are the first group of callbacks to fire. 处理created事件非常有用，可以给模板实例增加值，然后在模板helper中使用Template.instance()获取。

```
Template.myPictures.onCreated(function () {
  // set up local reactive variables
  this.highlightedPicture = new ReactiveVar(null);

  // register this template within some central store
  GalleryTemplates.push(this);
});
```

```
Template.myTemplate.onDestroyed		client
注册一个函数，当模板实例从DOM中移除并清理时调用

参数
	callback 	Function
```

当模板实例由于任何原因从页面中移除且不是由重新渲染替换时，调用回调函数。在回调函数内部，this指向模板实例。

This group of callbacks is most useful for cleaning up or undoing any external effects of created or rendered groups. This group fires once and is the last callback to fire.

```
Template.myPictures.onDestroyed(function () {
  // deregister from some central store
  GalleryTemplates = _.without(GalleryTemplates, this);
});
```
##模板实例
可以给模板实例增加属性，在模板reactive 更新时，属性不会丢失。

除了下面介绍的属性和方法，你可以随意的给模板实例增加属性。

只能在onRendered 回调中使用findAll,find,firstNode 和lastNode，在onCreated和onDestroyed 回调中不行，因为需要模板实例存在于DOM中。

模板实例对象是 instanceof Blaze.TemplateInstance

```
template.findAll(selector)		client
在模板实例中找到所有匹配selector的元素
```

返回DOM元素的数组

```
template.$(selector)		client
在模板实例中找到所有匹配selector的元素，返回jQuery对象
```

```
template.find(selector)		client
在模板实例中找到一个匹配selector的元素。		
```

如果没有，返回null

```
template.firstNode		client
模板实例中第一个顶级DOM元素
```

```
template.lastNode		client
模板实例中最后一个顶级DOM元素
```

```
template.data		client
模板实例最后一次调用时的数据上下文
```

模板每次重新渲染，都会更新。只读，非reactive。

```
template.autorun(funFunc)		client
Tracker.autorun 的另一版本，当模板销毁时停止
参数
	runFunc	Function
	要运行的函数，接受一个参数，一个Tracker.Computation对象
```

可以在onRendered或onCreated 回调函数中reactive更新DOM或是模板实例。当模板被销毁时，computation自动被停止。

是template.view.autorun 的别名。

```
template.subscribe(name,[arg1,arg2...],[callbacks])   client
Meteor.subscribe 另一个版本，当模板销毁时订阅会被停止。

参数
	name	string
	订阅的name
	arg1,arg2...		any
	可选的参数，传递给服务端发布函数
	callback		Function or Object
	可选。可以包含onStop 和 onReady 回调函数。如果是一个函数而不是对象，则解释为onReady回调
```

可以在onCreated 回调中使用this.subscribe 来声明模板依赖的发布数据。当模板被销毁时，订阅会自动停止。

还有一个配套的函数 Template.instance().subscriptionsReady() ,当this.subscribe订阅的所有数据都准备好时，返回true。

在模板的HTML里，你可以使用内置helper Template.subscriptionsReady ,来做加载状态。

例如：

```
Template.notifications.onCreated(function () {
  // Use this.subscribe inside onCreated callback
  this.subscribe("notifications");
});
```

```
<template name="notifications">
  {{#if Template.subscriptionsReady}}
    <!-- This is displayed when all data is ready. -->
    {{#each notifications}}
      {{> notification}}
    {{/each}}
  {{else}}
    Loading...
  {{/if}}
</template>
```

一个订阅依赖数据上下文的例子：

```
Template.comments.onCreated(function () {
  var self = this;

  // Use self.subscribe with the data context reactively
  self.autorun(function () {
    var dataContext = Template.currentData();
    self.subscribe("comments", dataContext.postId);
  });
});

```

```
{{#with post}}
  {{> comments postId=_id}}
{{/with}}
```

```
template.view  	client
本次调用模板产生的View 对象
```

```
Template.registerHelper(name,function)	  client
定义一个可以从所有模板中使用的helper 函数
```

```
Template.instance()		client
对应当前模板helper，事件处理器，回调，或是autorun 的模板实例。如果没有，则为null
```

```
Template.currentData() 		client

- 在onCreated,onRendered,onDestroyed回调中，返回模板的数据上下文
- 在事件处理器中，返回事件处理器定义所在的模板的数据上下文
- 在helper中，返回helper 所在的DOM节点的数据上下文
```

```
Template.parentData([numLevels])		client
获取当前数据上下文的父级数据上下文

参数
	numLevels		Integer
	向上查询的级别，默认是1
	
```
例如：Template.parentData(0) 等于Template.currentData(). Template.parentData(2) 等于模板中的{{../..}}


```
Template.body		client
代表<body>的模板对象
```

可以给Template.body 定义helper和事件，就像其他Template.myTemplate对象一样。

给Template.body 定义的helper are only available in the <body> tags of your app。要想注册全局helper，使用Template.registerHelper. 给Template.body 定义的event map 不会应用到：通过Blaze.render,jQuery,或是DOM API 添加到body的元素，或是body元素本身。 用jQuery 或是DOM API处理body ,window,document上的事件。

```
{{>Template.dynamic template=template [data=data]}}    Templates
动态的插入模板

参数
	template	string
	要引入的模板名
	data		Object
	可选的，引入模板的数据上下文
```

Template.dynamic 允许你动态的引入模板，模板名可以是通过helper计算出来的，reactive 变化。如果没有提供data参数，则使用当前的数据上下文

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










