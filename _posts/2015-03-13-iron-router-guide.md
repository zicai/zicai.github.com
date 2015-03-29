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

## Template Lookup

如果在你的路由里没有指定template 选项，也米有指定渲染哪一个模板，router会自动根据路由的名字渲染模板。默认情况下，router会寻找class case name of the template。

例如，假设你定义了下面这样一个路由：

```javascript
Router.route('/items/:_id', {name: 'items.show'});
```


router默认会寻找名叫`ItemsShow`的模板，每个单词首字母大写，移除标点符号。如果你想要自定义该行为，你可以设置你自己的converter 函数。例如：假设你不想要任何转换。你可以像下面这样设置converter函数：

```
Router.setTemplateNameConverter(function (str) { return str; });
```

## Path and Link Template Helpers

### pathFor

有一些模板helper，可以基于路由来创建超链接。首先，我们使用`{{pathFor}}`helper 来创建一个基于路由的链接。基于上面的`post.show`路由我们可以创建链接：

```html
{{#with post}}
  <a href="{{pathFor route='post.show'}}">Post Show</a>
{{/with}}
```

假设我们有一个post ,id 为1，上面的代码等同于：

```html
<a href="/posts/1">Post Show</a>
```

我们可以传递`data`, `query` and `hash`选项给pathFor helper.

```html
<a href="{{pathFor route='post.show' data=getPost query='q=s' hash='frag'}}">Post Show</a>
```

data 对象会被插入到路由参数中。query和hash 参数会被追加到href作为query string 和hash fragment。假设data对象如下：

```javascript
data = { _id: 1 };
```

上面的`pathFor`表达式会生成类似下面的链接：

```html
<a href="/post/1?q=s#frag">Post Show</a>
```

使用`pathFor`helper的好处是我们不用在整个application里硬编码`href`属性

### urlFor

`pathFor` helper用来基于路由生成路径，`urlFor` 则会生成完整的URL。例如：`pathFor`生成一个路径`/posts/1`，`urlFor`就会生成`http://mysite.com/posts/1`。

### linkTo

`linkTo` helper根据给定的路由，参数，hash和query自动生成锚点标签。你甚至可以提供锚点里内容。

```html
{{#linkTo route="post.show" data=getData query="q=s" hash="hashFrag" class="my-cls"}}
  <span style="color: orange;">
    Post Show
  </span>
{{/linkTo}}
```

上面的表达式会生成类似下面的HTML：

```html
<a href="/posts/1?q=s#hashFrag" class="my-cls">
  <span style="color: orange;">
    Post Show
  </span>
</a>
```

## Route Options
到目前为止，你已经看到了一些可以提供给路由的选项，例如`name`选项。还有其他一些选项和几种不同的方式提供选项给路由。

### Route Specific Options

下面的例子中，我会忽略路由函数，而只提供选项对象。选项对象会解释每一个可以使用的选项。

```javascript
Router.route('/post/:_id', {
  // The name of the route.
  // Used to reference the route in path helpers and to find a default template
  // for the route if none is provided in the "template" option. If no name is
  // provided, the router guesses a name based on the path '/post/:_id'
  name: 'post.show',

  // To support legacy versions of Iron.Router you can provide an explicit path
  // as an option, in case the first parameter is actually a route name.
  // However, it is recommended to provide the path as the first parameter of the
  // route function.
  path: '/post/:_id',

  // If we want to provide a specific RouteController instead of an anonymous
  // one we can do that here. See the Route Controller section for more info.
  controller: 'CustomController',

  // If the template name is different from the route name you can specify it
  // explicitly here.
  template: 'Post',

  // A layout template to be used with this route.
  // If there is no layout provided, a default layout will
  // be used.
  layoutTemplate: 'ApplicationLayout',

  // A declarative way of providing templates for each yield region
  // in the layout
  yieldRegions: {
    'MyAside': {to: 'aside'},
    'MyFooter': {to: 'footer'}
  },

  // a place to put your subscriptions
  subscriptions: function() {
    this.subscribe('items');
    
    // add the subscription to the waitlist
    this.subscribe('item', this.params._id).wait();
  },

  // Subscriptions or other things we want to "wait" on. This also
  // automatically uses the loading hook. That's the only difference between
  // this option and the subscriptions option above.
  waitOn: function () {
    return Meteor.subscribe('post', this.params._id);
  },

  // A data function that can be used to automatically set the data context for
  // our layout. This function can also be used by hooks and plugins. For
  // example, the "dataNotFound" plugin calls this function to see if it
  // returns a null value, and if so, renders the not found template.
  data: function () {
    return Posts.findOne({_id: this.params._id});
  },

  // You can provide any of the hook options described below in the "Using
  // Hooks" section.
  onRun: function () {},
  onRerun: function () {},
  onBeforeAction: function () {},
  onAfterAction: function () {},
  onStop: function () {},

  // The same thing as providing a function as the second parameter. You can
  // also provide a string action name here which will be looked up on a Controller
  // when the route runs. More on Controllers later. Note, the action function
  // is optional. By default a route will render its template, layout and
  // regions automatically.
  // Example:
  //  action: 'myActionFunction'
  action: function () {
    // render all templates and regions for this route
    this.render();
  }
});
```

### Global Default Options
上面所有的选项都可以给Router设置，也就相当于设置了所有路由的默认选项。使用`configure`方法设置默认路由选项。

```javascript
Router.configure({
  layoutTemplate: 'ApplicationLayout',

  template: 'DefaultTemplate'

  // .
  // .
  // .
});
```


在路由里声明的选项会覆盖默认的Router的选项。

## Subscriptions
有时候你可能会想等待一个或多个订阅准备好，或是其他一些动作的执行结果。例如，你想在等待订阅数据的时候显示一个加载模板。

### Wait and Ready
你可以用`wait`方法把一个订阅加到wait list中去。当wait list中所有item都已准备好时，调用`this.ready()`会返回true。

```javascript
Router.route('/post/:_id', function () {
  // add the subscription handle to our waitlist
  this.wait(Meteor.subscribe('item', this.params._id));

  // this.ready() is true if all items in the wait list are ready

  if (this.ready()) {
    this.render();
  } else {
    this.render('Loading');
  }
});
```

上面例子的另一种写法就是：直接调用订阅的`wait`方法。在这种情况下，你需要用`this.subscribe`而不是`Meteor.subscribe`。

```javascript
Router.route('/post/:_id', function () {
  this.subscribe('item', this.params._id).wait();

  if (this.ready()) {
    this.render();
  } else {
    this.render('Loading');
  }
});
```

### The subscriptions Option
使用路由的`subscriptions`选项：

```
Router.route('/post/:_id', {
  subscriptions: function() {
    // returning a subscription handle or an array of subscription handles
    // adds them to the wait list.
    return Meteor.subscribe('item', this.params._id);
  },

  action: function () {
    if (this.ready()) {
      this.render();
    } else {
      this.render('Loading');
    }
  }
});
```

Your `subscriptions` function can return a single subscription handle (the result of `Meteor.subscribe`) or an array of them. The subscription(s) will be used to drive the `.ready()` state. 

You can also inherit subscriptions from the global router config or from a controller (see below).

### The waitOn Option
Another alternative is to use `waitOn` instead of `subscribe`. This has the same effect but automatically short-circuits your route action and any before hooks (see below), and renders a `loadingTemplate` instead. You can specify that template on the route or the router itself:

```javascript
Router.route('/post/:_id', {
  // this template will be rendered until the subscriptions are ready
  loadingTemplate: 'loading',
  
  waitOn: function () {
    // return one handle, a function, or an array
    return Meteor.subscribe('post', this.params._id);
  },

  action: function () {
    this.render('myTemplate');
  }
});
```

## Server Routing

### Creating Routes
So far you've seen features mostly intended for the browser. But you can also
create server routes with full access to the NodeJS request and response
objects. To create a server route you provide the `where: 'server'` option to
the route.

```javascript
Router.route('/download/:file', function () {
  // NodeJS request object
  var request = this.request;

  // NodeJS  response object
  var response = this.response;

  this.response.end('file download content\n');
}, {where: 'server'});
```

### Restful Routes
You can even create server-side restful routes which correspond to an http verb.
This is particularly useful if you're setting up a webhook for another service
to post data to.

```javascript
Router.route('/webhooks/stripe', { where: 'server' })
  .get(function () {
    // GET /webhooks/stripe
  })
  .post(function () {
    // POST /webhooks/stripe
  })
  .put(function () {
    // PUT /webhooks/stripe
  })
```

### 404s and Client vs Server Routes
When you initially navigate to your Meteor application's url, the server router
will see if there are any routes defined for that url, either on the server or
on the client. If no routes are found, the server will send a 404 http status
code to indicate no resource was found for the given url.


## Plugins
Plugins are a way to reuse functionality in your router, either that you've
built for your own applications, or from other package authors. There's even a
built-in plugin called "dataNotFound".

To use a plugin just call the `plugin` method of Router and pass the name of the
plugin and any options for the plugin.

```javascript
Router.plugin('dataNotFound', {notFoundTemplate: 'notFound'});
```

This out-of-box plugin will automatically render the template named "notFound" 
if the route's data is falsey (i.e. `! this.data()`).

### Applying Plugins to Specific Routes
You can apply a plugin to a specific route by passing an `except` or `only` option
to the respective plugin function. This is useful for server routes, where you
explicitly don't want to run plugins designed for the client.

```javascript
Router.plugin('dataNotFound', {
  notFoundTemplate: 'NotFound', 
  except: ['server.route']
  // or only: ['routeOne', 'routeTwo']
});
```

In the above example, the dataNotFound will be applied to all routes except the 
route named 'server.route'.

### Creating Plugins
To create a plugin just put your function on the `Iron.Router.plugins` object
like this:

```javascript
Iron.Router.plugins.loading = function (router, options) {
  // this loading plugin just creates an onBeforeAction hook
  router.onBeforeAction('loading', options);
};
```
The plugin function will get called with the router instance and any options the
user passed.

*Package authors are encouraged to create new plugins!*

## Hooks

### Using Hooks

一个钩子就是一个函数。钩子提供了一种可以介入路由执行过程的方式，通常是自定义渲染行为或是执行一些业务逻辑。

下面的例子，我们的目的是如果用户已登录，渲染模板。通过使用`onBeforeAction`方法添加一个钩子，告诉Router，我们想让这个函数在路由函数之前执行或是"action"函数。

```javascript
Router.onBeforeAction(function () {
  // all properties available in the route function
  // are also available here such as this.params

  if (!Meteor.userId()) {
    // if the user is not logged in, render the Login template
    this.render('Login');
  } else {
    // otherwise don't hold up the rest of hooks or our route/action function
    // from running
    this.next();
  }
});
```


假设我们有下面这样一个路由：

```javascript
Router.route('/admin', function () {
  this.render('AdminPage');
});
```

当用户导航到"/admin"时，我们的onBeforeAction钩子函数会在路由函数执行之前运行。如果用户没有登录，那么路由函数永远不会执行，`AdminPage`也就不会被渲染。

钩子函数和所有在调度路由时运行的函数都运行在**reactive computation**：如果任何reactive 数据源invalidate the computation，它们都会重新执行。在上面的例子中，如果`Meteor.user()`发生改变，那么整个路由函数都会重新执行。

### Applying Hooks to Specific Routes
可以通过传递`except`或者`only`选项给钩子来指定它到特定的路由。

```javascript
Router.onBeforeAction(myAdminHookFunction, {
  only: ['admin']
  // or except: ['routeOne', 'routeTwo']
});
```

上面的例子，myAdminHookFunction只会应用到名为`admin`的路由

### Using the Iron.Router.hooks Namespace

Package作者可以添加钩子函数到`Iron.Router.hooks`，使用者可以通过字符串名来引用。

```javascript
Iron.Router.hooks.customPackageHook = function () {
  console.log('hi');
  this.next();
};

Router.onBeforeAction('customPackageHook');
```
### Available Hook Methods

* **onRun**: Called when the route is first run. It is not called again if the
  route reruns because of a computation invalidation. This makes it a good 
  candidate for things like analytics where you want be sure the hook only runs once. Note that this hook *won't* run again if the route is reloaded via hot code push.
  当路由第一次执行时调用。如果因为computation invalidation 导致路由重新执行，它不会被调用。

* **onRerun**:
  如果因为computation invalidation 导致路由重新执行时被调用。

* **onBeforeAction**: Called before the route or "action" function is run. These
  hooks behave specially. If you want to continue calling the next function you
  *must* call `this.next()`. If you don't, downstream onBeforeAction hooks and
  your action function will *not* be called.

* **onAfterAction**: Called after your route/action function has run or had a
  chance to run. These hooks behave like normal hooks and you don't need to call
  `this.next()` to move from one to the next.

* **onStop**: Called when the route is stopped, typically right before a new
  route is run.


### Server Hooks and Connect

On the server, the API signature for a `onBeforeAction` hook is identical to that of a [connect](https://github.com/senchalabs/connect) middleware:

```
Router.onBeforeAction(function(req, res, next) {
  // in here next() is equivalent to this.next();
}, {where: 'server'});
```

This means you can attach any connect middleware you like on the server side using `Router.onBeforeAction()`. For convience, IR makes express' [body-parser](https://github.com/expressjs/body-parser) available at `Iron.Router.bodyParser`.

The Router attaches the JSON body parser automatically.

## Route Controllers

当Router处理一个URL时，它会创建一个`Iron.RouteController`对象。`RouteController`给我们提供了一个保存状态的地方，直到另一个路由运行。

在路由函数中我们调用了一些方法，例如：`this.render()`和`this.layout()`。其中，`this`对象实际上就是`RouteController`的实例。如果你构建的只是一个很简单的应用，那你无需考虑`RouteController`。但是如果你的应用体积越来越大，直接使用`RouteControllers`会带来两点好处：

- **继承**：你可以从其它RouteController继承to model your
  application's behavior.
- **组织**：你可以把路由逻辑放到RouteController文件中去，而不是把所有业务逻辑放到一个超大路由文件中。


### Creating Route Controllers

用下面的方式创建一个自定义`RouteController`:

```javascript
PostController = RouteController.extend();
```


当你定义一个路由的时候，你可以指定一个要使用的controller，否则Router会尝试基于路由的名字自动寻找controller。

```javascript
Router.route('/post/:_id', {
  name: 'post'
});
```

上面定义的路由会自动寻找`PostController`。我们可以通过提供一个controller 选项来告诉路由使用另外一个controller。

```javascript
Router.route('/post/:_id', {
  name: 'post.show',
  controller: 'PostController'
});
```

路由里可以使用的选项在我们的`RouteControllers`里都可以使用

```javascript
PostController = RouteController.extend({
  layoutTemplate: 'PostLayout',

  template: 'Post',

  waitOn: function () { return Meteor.subscribe('post', this.params._id); },

  data: function () { return Posts.findOne({_id: this.params._id}) },

  action: function () {
    this.render();
  }
});
```

我们可能会把一些选项用`Router.configure`定义为全局，把一些选项定义到`Route`，还有一些定义在`RouteController`。Iron.Router会按照下面的的顺序查找选项：

1. Route
2. RouteController
3. Router


### Inheriting from Route Controllers

RouteController可以从其他的RouteController继承。这使得你可以很好地组织你的应用。

假设我们有一个`ApplicationController`作为所有路由的父类。

```javascript
ApplicationController = RouteController.extend({
  layoutTemplate: 'ApplicationLayout',

  onBeforeAction: function () {
    // do some login checks or other custom logic
    this.next();
  }
});

Router.configure({
  // this will be the default controller
  controller: 'ApplicationController'
});

// now we have a route for posts
Router.route('/posts/:_id', {
  name: 'post'
});

// inherit from `ApplicationController` and override any
// behavior you'd like.
PostController = ApplicationController.extend({
  layoutTemplate: 'PostLayout'
});
```

*注意：由于你无法精确地控制文件加载顺序，所以你需要保证父RouteController优先于子RouteController加载*

### Accessing the Current Route Controller

有两种方式可以获取当前的`RouteController`。

如果在客户端，你可以使用`Router.current()`方法。它会reactively 返回当前`RouteController`的实例。记住，如果路由还没执行，这个值可能为`null`。

在template helper里可以用`Iron.controller()`方法获取当前的`RouteController`

```javascript
Router.route('/posts', function () {
  this.render('Posts');
});
```

上面的路由会渲染下面定义的`Posts`模板。

```html
<template name="Posts">
  Posts
</template>
```


假设我们想要从一个template helper里获取当前的controller。

```javascript
Template.Posts.helpers({
  myHelper: function () {
    var controller = Iron.controller();

    // now we can get properties and call methods on the controller
  }
});
```

### Setting Reactive State Variables
You can set reactive state variables on controllers using the `set` method on
the controller's [ReactiveDict](https://atmospherejs.com/meteor/reactive-dict) `state`.

Let's say we want to store the post `_id` in a reactive variable:

```javascript
Router.route('/posts/:_id', {name: 'post'});

PostController = RouteController.extend({
  action: function () {
    // set the reactive state variable "postId" with a value
    // of the id from our url
    this.state.set('postId', this.params._id);
    this.render();
  }
});
```

### Getting Reactive State Variables
You can get a reactive variable value by calling `this.state.get("key")` on the
`RouteController`. Using the example above, let's grab the value of `postId`
from a template helper.

```javascript
Template.Post.helpers({
  postId: function () {
    var controller = Iron.controller();

    // reactively return the value of postId
    return controller.state.get('postId');
  }
});
```

## Custom Router Rendering
So far we've been letting the Router render itself to the page automatically.
But you can also control precisely where the Router renders itself by using a
global helper method.

```html
<body>
  <h1>Some App Html</h1>
  <div class="container">
    {{! Render the router into this div instead of the body}}
    {{> Router}}
  </div>
</body>
```

## Legacy Browser Support
Legacy browsers do not support the HTML5 `pushState` and `history` features
required for normal client side browsing with the `Router`. To solve this
problem, the `Router` can fall back to using hash fragments in the url.
Actually, under the hood, `iron-router` uses a package called `iron-location`
which handles all of this. It works similarly to the `History.js` project but
works seamlessly.

This functionality is automatically enabled for **IE8** and **IE9**. If you want
to enable it manually to play around you can configure `Iron.Location` like
this:

```javascript
Iron.Location.configure({useHashPaths: true});
```

Even though the url will appear differently in the browser when using this mode,
the url, query, hash and parameters will look like their regular values inside
of `RouteController` functions. Here are a few examples of how urls will be
translated.

```
http://localhost:3000/items/5?q=s#hashFrag
```

The url above would be transformed to the url below in your browser.

```
http://localhost:3000/#/items/5?q=s&__hash__=hashFrag
```

But in your `RouteController` functions you can access the url, query and hash
values just like you have before.

```
Router.route('/items/:_id', function () {
  var id = this.params._id; // "5"
  var query = this.params.query; // {q: "s"}
  var hash = this.params.hash; // "hashFrag"
});
```

**NOTE: Please let us know if you can help test support on other browsers!**