---
layout: post
category : lessons
title: "Iron router guide"
tagline: "Supporting tagline"
tags : []
---

既可以工作在服务器也能工作在浏览器的路由，专门为meteor设计。

## 快速开始

你可以通过Meteor的包管理系统来安装iron:router

```bash
> meteor add iron:router
```

用`meteor update`可以将iron:router更新到最新版：



```bash
> meteor update iron:router
```

在你的javascript文件中创建一条路由。默认，路由是给客户端创建的，运行在浏览器中。

```javascript
Router.route('/', function () {
  this.render('Home');
});
```

当用户导航到路径"/",上面的路由就会渲染名字叫"Home"的模板到页面上。

```javascript
Router.route('/items');
```

上面的路由会自动渲染名字为"Items" 或 "items"的模板到页面里。在这样简单地例子中，你都无需提供一个路由函数。

目前为止，我们只创建了直接在浏览器运行的路由。其实，我们还可以创建服务器端路由。These hook directly into the HTTP request and
are used to implement REST endpoints.

```javascript
Router.route('/item', function () {
  var req = this.request;
  var res = this.response;
  res.end('hello from the server\n');
}, {where: 'server'});
```


`where: 'server'`选项告诉Router，这是一个服务器端路由。

## 内容列表

- [Concepts](#concepts)
  - [Server Only](#server-only)
  - [Client Only](#client-only)
  - [Client and Server](#client-and-server)
  - [Reactivity](#reactivity)
- [Route Parameters](#route-parameters)
- [Rendering Templates](#rendering-templates)
- [Rendering Templates with Data](#rendering-templates-with-data)
- [Layouts](#layouts)
  - [Rendering Templates into Regions with JavaScript](#rendering-templates-into-regions-with-javascript)
  - [Setting Region Data Contexts](#setting-region-data-contexts)
  - [Rendering Templates into Regions using contentFor](#rendering-templates-into-regions-using-contentfor)
- [Client Navigation](#client-navigation)
  - [Using Links](#using-links)
  - [Using JavaScript](#using-javascript)
  - [Using Redirects](#using-redirects)
  - [Using Links to Server Routes](#using-links-to-server-routes)
- [Named Routes](#named-routes)
- [Template Lookup](#template-lookup)
- [Path and Link Template Helpers](#path-and-link-template-helpers)
  - [pathFor](#pathfor)
  - [urlFor](#urlFor)
  - [linkTo](#linkTo)
- [Route Options](#route-options)
  - [Route Specific Options](#route-specific-options)
  - [Global Default Options](#global-default-options)
- [Subscriptions](#subscriptions)
  - [Wait and Ready](#wait-and-ready)
  - [The subscriptions Option](#the-subscriptions-option)
  - [The waitOn Option](#the-waiton-option)
- [Server Routing](#server-routing)
  - [Creating Routes](#creating-routes)
  - [Restful Routes](#restful-routes)
  - [404s and Client vs Server Routes](#404s-and-client-vs-server-routes)
  - [Server Middleware and Connect](#server-middleware-and-connect)
- [Plugins](#plugins)
  - [Creating Plugins](#creating-plugins)
- [Hooks](#hooks)
  - [Using Hooks](#using-hooks)
  - [Applying Hooks to Specific Routes](#applying-hooks-to-specific-routes)
  - [Using the Iron.Router.hooks Namespace](#using-the-iron-router-hooks-namespace)
  - [Available Hook Methods](#available-hook-methods)
- [Route Controllers](#route-controllers)
  - [Creating Route Controllers](#creating-route-controllers) 
  - [Inheriting from Route Controllers](#inheriting-from-route-controllers) 
  - [Accessing the Current Route Controller](#accessing-the-current-route-controller)
  - [Setting Reactive State Variables](#setting-reactive-state-variables)
  - [Getting Reactive State Variables](#getting-reactive-state-variables)
- [Custom Router Rendering](#custom-router-rendering)
- [Legacy Browser Support](#legacy-browser-support)

## Concepts

### Server Only

在一个典型的web app里，你发起一个HTTP请求，请求一个特定的URL，发送到服务器，例如："/items/5"，服务器端路由根据URL决定调用哪一个函数。这个函数很可能是返回一些HTML，然后关闭链接。

### Client Only

在一些新型的web app 里，你会用到客户端路由，例如pagejs 或者Backbone路由。这些路由在浏览器里运行，使你可以在app里导航而不用等待服务器响应。通常他们都是利用HTML5特性，例如：pushState或是url hash fragments。

### Client and Server

Iron.Router运行在客户端和服务器端。你可以定义只在服务器端运行的路由，也可以定义只在客户端运行的路由。这使得你的app在加载完之后响应速度很快，因为无需重新加载整个页面。

router知晓所有在服务器端和客户端的路由。这意味着你可以点击一个链接，他可能把你带到一个服务器端路由，也可能是一个客户端路由。如果两者都没有定义，就可以返回一个404页面到客户端。

### Reactivity
路由函数和大多数hooks运行在reactive computation。也就是说当一个reactive data source发生变化时，他们会自动重新运行。例如：如果你在路由函数中调用了`Meteor.user()` ，每次`Meteor.user()` 的值发生变化时，路由函数都会重新执行。

## Route Parameters
路由可以有变量参数。例如，你可以创建一个用于展示带ID的文章的路由
。`id`是一个变量，取决于你想看的文章，例如："/posts/1" 或是 "/posts/2"。用`:`在路由里声明命名参数，后面跟着参数名。当用户导航到那个URL时，真实的参数值会保存为路由函数里`this.params`的一个属性。

在下面的例子中，我们有一个叫做`_id`的路由参数。如果我们导航到`/post/5`，在路由函数中我们可以通过`this.params._id`得到`_id`的真实值。也就是`this.params._id => 5`。

```javascript
// given a url like "/post/5"
Router.route('/post/:_id', function () {
  var params = this.params; // { _id: "5" }
  var id = params._id; // "5"
});
```

你可以定义多个路由参数。下面的例子中，有一个`_id`，还有一个`commentId`。如果你导航到`/post/5/comments/100`，那么在你的路由函数中`this.params._id => 5`
同时 `this.params.commentId => 100`

```javascript
// given a url like "/post/5/comments/100"
Router.route('/post/:_id/comments/:commentId', function () {
  var id = this.params._id; // "5"
  var commentId = this.params.commentId; // "100"
});
```

如果URL里包含query string或是hash fragment，那你可以通过`this.params`对象的`query`属性和`hash`属性来获取。

```javascript
// given the url: "/post/5?q=s#hashFrag"
Router.route('/post/:_id', function () {
  var id = this.params._id;
  var query = this.params.query;
  
  // query.q -> "s"
  var hash = this.params.hash; // "hashFrag"
});
```

**注意**:如果你想当hash变化时重新运行一个函数，你可以用下面的方法：

```javascript
// get a handle for the controller.
// in a template helper this would be
// var controller = Iron.controller();
var controller = this;

// reactive getParams method which will invalidate the comp if any part of the params change
// including the hash.
var params = controller.getParams();
```

By default the router will follow normal browser behavior. If you click a link with a hash frag it will scroll to an element with that id. If you want to use `controller.getParams()` you can put that in either your own autorun if you want to do something procedural, or in a helper.



## Rendering Templates

通常当用户导航到一个特定URL时，我们想渲染一个模板。例如：当用户导航到`/posts/1`时，我们想渲染名叫`Post`的模板：

```html
<template name="Post">
  <h1>Post: {{title}}</h1>
</template>
```

```javascript
Router.route('/post/:_id', function () {
  this.render('Post');
});
```

在路由函数中，可以通过调用`render`方法来渲染一个模板。`render`方法把模板名作为第一个参数。

## Rendering Templates with Data

在上面的例子中，`title`的值没有定义。e我们可以给post模板创建一个
叫做`title`的helper，或者直接在路由函数中给模板设置data context。在调用`render`时，提供一个`data`选项作为第二个参数。

```javascript
Router.route('/post/:_id', function () {
  this.render('Post', {
    data: function () {
      return Posts.findOne({_id: this.params._id});
    }
  });
});
```

## Layouts

Layout 使得你可以在多个页面中共用同样的外观和风格，而不必在每个单独的页面中拷贝相同的HTML和逻辑。

Layout也是模板。但是在Layout里你可以使用一个特殊的helper叫做`yield`。你可以把`yield`看做一个内容的占位符。我们把占位符叫做*region*。当路由运行时，内容会被插入到region里。这使得我们可以在多个页面中共用Layout，只是更改yield regions。

```html
<template name="ApplicationLayout">
  <header>
    <h1>{{title}}</h1>
  </header>

  <aside>
    {{> yield "aside"}}
  </aside>

  <article>
    {{> yield}}
  </article>

  <footer>
    {{> yield "footer"}}
  </footer>
</template>
```


在路由函数中，可以通过调用`layout`方法来指明使用哪个Layout。

```javascript
Router.route('/post/:_id', function () {
  this.layout('ApplicationLayout');
});
```

如果你想给所有路由指定一个默认的Layout模板，可以配置全局Router选项。

```javascript
Router.configure({
  layoutTemplate: 'ApplicationLayout'
});
```

### Rendering Templates into Regions with JavaScript

在路由函数中，我们可以告诉router哪一个region对应哪个模板。

```html
<template name="Post">
  <p>
    {{post_content}}
  </p>
</template>

<template name="PostFooter">
  Some post specific footer content.
</template>

<template name="PostAside">
  Some post specific aside content.
</template>
```

假设我们使用`ApplicationLayout` ，同时把上面定义的模板渲染到对应的region里。在路由函数中可以直接使用`render`方法的`to`选项。

```javascript
Router.route('/post/:_id', function () {
  // use the template named ApplicationLayout for our layout
  this.layout('ApplicationLayout');

  // render the Post template into the "main" region
  // {{> yield}}
  this.render('Post');

  // render the PostAside template into the yield region named "aside" 
  // {{> yield "aside"}}
  this.render('PostAside', {to: 'aside'});

  // render the PostFooter template into the yield region named "footer" 
  // {{> yield "footer"}}
  this.render('PostFooter', {to: 'footer'});
});
```

### Setting Region Data Contexts

可以给`render`方法提供`data`选项来给每个region指定data context。还可以给整个Layout指定一个data context。

```javascript
Router.route('/post/:_id', function () {
  this.layout('ApplicationLayout', {
    data: function () { return Posts.findOne({_id: this.params._id}) }
  });

  this.render('Post', {
    // we don't really need this since we set the data context for the
    // the entire layout above. But this demonstrates how you can set
    // a new data context for each specific region.
    data: function () { return Posts.findOne({_id: this.params._id})
  });

  this.render('PostAside', {
    to: 'aside',
    data: function () { return Posts.findOne({_id: this.params._id})
  });

  this.render('PostFooter', {
    to: 'footer',
    data: function () { return Posts.findOne({_id: this.params._id})
  });
});
```

### Rendering Templates into Regions using contentFor

在路由函数里把模板渲染到region里是非常有用的，尤其是当我们需要执行一些自定义逻辑或是模板的名字是动态的。但是还有一种更简便的给region提供内容的方式：直接在主模板里使用`contentFor`helper。假设我们使用仍然使用上面例子中的`ApplicationLayout`。但是这次，我们不会给每一个region定义一个新模板，而是在`Post`模板中提供content inline。

```html
<template name="Post">
  <p>
    {{post_content}}
  </p>

  {{#contentFor "aside"}}
    Some post specific aside content.
  {{/contentFor}}

  {{#contentFor "footer"}}
    Some post specific footer content.
  {{/contentFor}}
</template>
```

现在我们只需指定Layout，然后只渲染`Post`模板，而不用指明每一个独立的region

```javascript
Router.route('/post/:_id', function () {
  this.layout('ApplicationLayout', {
    data: function () { return Posts.findOne({_id: this.params._id}) }
  });

  // this time just render the template named "Post" into the main
  // region
  this.render('Post');
});
```

你甚至可以提供一个模板选项给`contentFor`helper，替代提供inline block content。

```html
<template name="Post">
  <p>
    {{post_content}}
  </p>

  {{> contentFor region="aside" template="PostAside"}}

  {{> contentFor region="footer" template="PostFooter"}}
</template>
```

## Client Navigation

大多数时间用户都是在你的App里导航，而无需发起请求到服务器。导航的方式有下面几种
### Using Links
通过点击超链接。假设我们使用下面的Layout，它包含几个超链接。
```html
<template name="ApplicationLayout">
  <nav>
    <ul>
      <li>
        <a href="/">Home</a>
      </li>
      
      <li>
        <a href="/one">Page One</a>
      </li>

      <li>
        <a href="/two">Page Two</a>
      </li>
    </ul>
  </nav>

  <article>
    {{> yield}}
  </article>
</template>

<template name="Home">
  Home
</template>

<template name="PageOne">
  Page One
</template>

<template name="PageTwo">
  Page Two
</template>
```

接下来，我们定义该页面的一些路由。

```javascript
Router.route('/', function () {
  this.render('Home');
});

Router.route('/one', function () {
  this.render('PageOne');
});

Router.route('/two', function () {
  this.render('PageTwo');
});
```

当应用第一次加载根路径`/`，会执行第一条路由，"Home"模板被渲染。

如果用户点击`Page One`链接，浏览器地址变成'/one',会执行第二条路由，"PageOne"模板会被渲染。

同样的，如果用户点击`Page Two`链接，浏览器地址变为'/two'，会执行第三条路由，"PageTwo"模板会被渲染。

即使浏览器地址发生了改变，但由于这是客户端路由，浏览器也不用发起请求到服务器。

### Using JavaScript

在Javascript里使用`Router.go`方法，你可以导航到一个给定的地址，甚至是一个路由的名子。假设我们给一个按钮定义了点击事件：

```html
<template name="MyButton">
  <button id="clickme">Go to Page One</button>
</template>
```

再点击事件处理函数中，我们告诉路由导航到`/one`

```javascript
Template.MyButton.events({
  'click #clickme': function () {
    Router.go('/one');
  }
});
```

这会将浏览器地址改变为`/one`，然后执行对应的路由。

### Using Redirects

在路由函数中使用`redirect`方法，可以从一个路由跳转到另一个路由。

```javascript
Router.route('/one', function () {
  this.redirect('/two');
});

Router.route('/two', function () {
  this.render('PageTwo');
});
```

### Using Links to Server Routes
假设你定义了一个服务器端路由。例如，一个文件下载路由必须通过服务器。
```javascript
Router.route('/download/:filename', function () {
  this.response.end('some file content\n');
}, {where: 'server'});
```

在HTML里就可以用这个链接下载指定文件。

```html
<a href="/download/myfilename">Download File</a>
```

当用户点击`Download File`链接时，router会发送请求到服务器，然后执行服务器端路由。

## Named Routes

路由可以有自己的名字，用名字就可以引用路由。如果你没有指定名字，路由会基于路径猜测名字。但是你可以通过`name`选项明确提供一个名字。

```javascript
Router.route('/posts/:_id', function () {
  this.render('Post');
}, {
  name: 'post.show'
});
```

命名完之后，就可以用下面的方式获取路由对象：

```javascript
Router.routes['post.show']
```


还可以在`Router.go`方法里使用路由名字：

```javascript
Router.go('post.show');
```

同事还可以提供一个参数对象，query和hash fragment 选项。

```javascript
Router.go('post.show', {_id: 1}, {query: 'q=s', hash: 'hashFrag'});
```

上面的代码会导航到：

```html
/post/1?q=s#hashFrag
```