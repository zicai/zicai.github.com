---
layout: post
category : lessons
title: "meteor deps"
tagline: "Supporting tagline"
tags : [meteor]
---

原文地址：(http://manual.meteor.com/)[http://manual.meteor.com/]
第一部分是关于Deps，一个小而强大的库，使Meteor可以transparent reactive programing。
第二部分是关于Blaze，Meteor的reactive 模板系统。

未来还会包括DDP(Distributed Data Protocol)和Unibuild(一个构建工具和打包系统)

##0. 总览
Meteor Deps 是一个非常小(~1k)但是非常强大的库，用来在javascript中进行transparent reactive programing.

##1. transparent reactive programing

我们在写软件的时候，尤其是应用软件，经常会有这样的需求：监视一些值(例如：数据库记录、表格当前选中的item，或是窗口的宽度，或是当前时间)，当它们发生变化时更新另外一些东西。

通常有几种形式来实现：
- poll and diff:周期性的获取要监视的值，如果发生变化，则执行一些操作
- Events:要监视的值发生变化时会触发一个事件。程序的其它部分(通常叫做controller)会安排监听事件，当事件被触发时，获取当前的值并执行其它更新。
- Bindings:

另外一种形式，还没有普遍使用，但是非常适合复杂的现代应用，就是 reactive programing。一个reactive programing就像电子表格系统。程序员说：A3单元格的值应该等于B1-B6，然后乘上C4的值。电子表格就负责自动的modeling 单元格之间数据流,所以当B2发生变化时，A3会自动更新。

所以reactive programing是一种声明式编程，程序员只需说明：应该发生什么，而不像在命令式编程中需要说明：如何发生。

reactive programing非常适合构建用户界面。
- 如果使用poll and diff，UI延迟会很高。
- 如果使用events，
- 如果使用bindings

唯一的问题？历史上，对大多数在生产环境使用的人来说，reactive programing系统非常复杂并且运行缓慢。它要求把reactive 表达式模型化为树的节点，每发生一个变化都不得不进行代价高的dataflow 分析

##2. 一个简单实例
```
var favoriteFood = "apples";
var favoriteFoodDep = new Deps.Dependency;

var getFavoriteFood = function () {
  favoriteFoodDep.depend();
  return favoriteFood;
};

var setFavoriteFood = function (newValue) {
  favoriteFood = newValue;
  favoriteFoodDep.changed();
};

getFavoriteFood();
// "apples"
```

上面的代码把favoriteFood 变成了reactive value。

那么什么是“reactive value”，如何使用它呢？

```
var handle = Deps.autorun(function () {
  console.log("Your favorite food is " + getFavoriteFood());
});
// "Your favorite food is apples"

setFavoriteFood("mangoes");
// "Your favorite food is mangoes"
```

Deps.autorun会立即执行传递给他的函数。

并且，

注意：并没有注册任何时间，也没有建立任何bindings,就发生了。autorun简单地记录运行函数时访问到的reactive value，当任何值发生变化时，重新运行函数。

```
setFavoriteFood("peaches");
// "Your favorite food is peaches"
setFavoriteFood("bananas");
// "Your favorite food is bananas"
handle.stop();
setFavoriteFood("cake");
// (nothing printed)
```

就像setTimeout 或是setInterval，调用Deps.autorun 时，它返回一个handle，可用于后面停止autorun。

```
var getReversedFood = function () {
  return getFavoriteFood().split("").reverse().join("");
};
getReversedFood();
// "ekac"

var handle = Deps.autorun(function () {
  console.log("Your favorite food is " + getReversedFood() + " when reversed");
});
// Your favorite food is ekac when reversed
setFavoriteFood("pizza");
// Your favorite food is azzip when reversed

```
Autorun可以使任何javascript代码块reactive。

唯一需要了解Deps的是reactive value的提供者(例如：数据库library)和reactive value的消费者(例如：Blaze，React，famo.us这样的前端库)。其余部分的代码，根本无需知道reactive programing 系统的存在。

##3. 用transparent reactive programing构建用户界面

给library增加Deps support很容易。

可以使用ReactiveDict。他提供一个简单地字典对象，对象的get(key)和set(key)函数都是reactive的。

```
var forecasts = new ReactiveDict;
forecasts.set("san-francisco", "cloudy");
forecasts.get("san-francisco");
// "cloudy"

```
接下来，我们把三藩市的天气情况展示到屏幕上，并使它跟随天气的变化而变化。

```
$('body').html("The weather here is <span class='forecast'></span>!");
Deps.autorun(function () {
  $('.forecast').text(forecasts.get('san-francisco'));
});
// Page now says "The weather here is cloudy!"

forecasts.set("san-francisco", "foggy");
// Page updates to say "The weather here is foggy!"
```

实际上，你都不用写这么多代码。利用Meteor Blaze，用下面的模板语法：

```
<template name="weather">
  The weather here is {{forecast}}!
</template>

// In app.js
Template.weather.forecast = function () {
  return forecasts.get("san-francisco");
};
```

Blaze解析模板，给必要的元素建立起autorun。基本的，Blaze做的很简单，让你可以在HTML模板里表达用户界面。

更复杂些的例子：

```
var forecasts = new ReactiveDict;
forecasts.set("Chicago", "cloudy");
forecasts.set("Tokyo", "sunny");

var settings = new ReactiveDict;
settings.set("city", "Chicago");

$('body').html("The weather in <span class='city'></span> is <span class='weather'></span>.");
Deps.autorun(function () {
  console.log("Updating");
  var currentCity = settings.get('city');
  $('.city').text(currentCity);
  $('.weather').text(forecasts.get(currentCity).toUpperCase());
});
// Prints "Updating"
// Page now says "The weather in Chicago is CLOUDY."
```

用Deps写的APP，代码可以分成三部分：
- reactive 数据源
- reactive 数据消费者
- 应用逻辑

##4. Deps 如何工作
一句话总结就是：Deps.autorun 记录在执行autorun函数时获取到的所有reactive value，当这些值中任何一个发生变化，就会触发一个事件导致autorun重新运行。

更精确的说：
每次调用Deps.autorun都会创建一个computation对象。无论何时autorun执行代码，全局变量Deps.currentComputation 就被设置为当前的Computation。当没有autorun执行时，则为null。

一个reactive数据源（例如：Reactive的get()方法）检查是否存在currentComputation,也就是说，是否在一个autorun中被调用。如果是，它会保持一个引用到当前的Computation，以后当相关数据(例如：)变化时安排在computation上调用方法。

换句话说：Deps简单地提供一个全局变量，作为可能发生变化的数据和需要知道数据何时发生变化的代码的汇合点。全局变量是reactive数据源指明当数据发生变化时通知哪一个autorun。

接下来的部分详细的介绍该系统

##5. 监控reactive value

###autorun可以嵌套
```
var weather = new ReactiveDict;

weather.set("sky", "sunny");
weather.set("temperature", "cool");

var weatherPrinter = Deps.autorun(function () {
  console.log("The sky is " + weather.get("sky"));
  var temperaturePrinter = Deps.autorun(function () {
    console.log("The temperature is " + weather.get("temperature"));
  });
});
// "The sky is sunny"
// "The temperature is cool"
```

存在嵌套autorun时，依赖属于最近的autorun。也就是说：改变内层的依赖，不会重新运行外层。而改变外层的依赖时，会重新运行外层autorun，这回销毁并重建内层的autorun。

尽管每次外层autorun运行时都会重建内层autorun。你也无需调用内层autorun的stop()方法。这是因为有自动cleanup约定。

什么时候利用嵌套autorun呢？像Blaze这种需要细粒度更新的库。用autorun作为reactive的边界

###自动cleanup 约定

如果一个库完全符合Deps,它必须遵循约定：如果一个在autorun里面创建的对象，需要stoped 或是cleaned up，当autorun重新运行时它应该自动自动stopped 。为什么这是正确的约定？大概是因为当重新运行时autorun会重建对象
###autorun何时抛出异常
当第一次执行的时候autorun抛出异常，异常会被传递到调用者。创建的Deps.computation 会被销毁，也不会重新执行。

如果是已经运行过一次，重新执行的时候抛出异常，会继续执行，不管后续rerun是否抛出异常。

###在Autorun内部获取当前的Computation
autorun函数被调用时会带一个参数：代表该autorun的Computation 对象

```
var partyInfo = new ReactiveDict;
partyInfo.set("status", "going-strong");

Deps.autorun(function (comp) {
  if (partyInfo.equals("status", "over")) {
    console.log("Mom's home! Get out! The party's over!");
    comp.stop();
    return;
  }

  runDiscoLights();
  crankTheMusic();
  eatPixieSticks();
});
```
如果status设置为over ,那么autorun会自己销毁。

下面的代码不行：

```
// WRONG
var partyRunner = Deps.autorun(function () {
  if (partyInfo.equals("status", "over")) {
    partyRunner.stop(); // WRONG
    return;
  }
  ...
})
```


###检测第一次执行

Computation对象的firstRun 属性指明是否是第一次执行。

```
var book = new ReactiveDict;
book.set("title", "A Game of Thrones");
Deps.autorun(function (c) {
  if (c.firstRun) {
    console.log("Curling up on the couch to read.");
  }
  console.log("Now reading: " + book.get("title"));
});
// "Curling up on the couch to read."
// "Now reading: A Game of Thrones"
```

###忽略特定reactive value的变化
通过Deps.nonreactive 来忽略特定reactive value的变化

```
var game = new ReactiveDict;
game.set("score", 42);
game.set("umpire", "Giraffe");

Deps.autorun(function () {
  Deps.nonreactive(function () {
    console.log("DEBUG: current game umpire is " + game.get("umpire"));
  });
  console.log("The game score is now " + game.get("score") + "!");
});
// "DEBUG: current game umpire is Giraffe"
// "The game score is now 42!"
```



##6. 细粒度reactivity

ReactiveDict : get VS equals

```
var data = new ReactiveDict;

// VERSION 1 (INEFFICIENT)
Deps.autorun(function () {
  if (data.get("favoriteFood") === "pizza")
    console.log("Inefficient code says: Time to get some pizza!");
  else
    console.log("Inefficient code say: No pizza for you!");
});

// VERSION 2 (MORE EFFICIENT)
Deps.autorun(function () {
  if (data.equals("favoriteFood", "pizza")) // CHANGED LINE
    console.log("Efficient code says: Time to get some pizza!");
  else
    console.log("Efficient code says: No pizza for you!");
});
```

粗放更新调用ReactiveDict的get("favoriteFood")，所以它依赖"favoriateFood"的值，只要"favoriteFood"发生变化，autorun就会重新运行。

细粒度更新调用ReactiveDict的equal("favoriteFood","pizza")，所以它不依赖"favoriteFood"的值，而是"favoriteFood"有一个特定值"pizza".

###Collection.find() VS Collection.cursor.fetch()

```
// VERSION 1 (INEFFICIENT)
Deps.autorun(function(){
  var allPosts = Posts.find().fetch();
  for (var post in allPosts) {
    if (post.tag === "kittens") console.log(post.title);
  }
});

// VERSION 2 (MORE EFFICIENT)
Deps.autorun(function(){
  var postsAboutKittens = Posts.find({tag:"kittens"}).fetch();
  postsAboutKittens.forEach(function (post) {
    console.log(post.title);
  });
});
```
inefficient版本依赖所有Post，efficient版本依赖标签为kittens的post

###Collection.cursor.count()

```
// VERSION 1: INEFFICIENT
Deps.autorun(function(){
  console.log(Posts.find({tags:"kittens"}).fetch().length);
});

// VERSION 2: EFFICIENT
Deps.autorun(function(){
  console.log(Posts.find({tags:"kittens"}).count());
});
```

如果在autorun里调用了cursor.count()，只有当document的数量发生变化时，才会rerun。

###细粒度和粗粒度更新指导

- 不要获取用不到的数据

##7. 创建Reactive value

###用ReactiveDict 库创建reactive value

```
var data = new ReactiveDict;

var getFavoriteFood = function () {
  return data.get("favoriteFood");
};

var setFavoriteFood = function (newValue) {
  data.set("favoriteFood", newValue);
};
```

限制是：ReactiveDict的值不能是对象或是数组，只能是string ，number，boolean ，Dates,null 和undefined

ReactiveDict 不包含在main deps 库中。可以通过 meteor add reactive-dict 来添加。

###一些例子

```
var data = new ReactiveDict;

data.set("now", new Date);
setTimeout(function () {
  data.set("now", new Date);
}, 1000);

var currentTime = function () {
  data.get("now");
};
```

窗口大小

```
var data = new ReactiveDict;

var updateWindowSize = function () {
  data.set("width", window.innerWidth);
  data.set("height", window.innerHeight);
};
updateWindowSize();
window.addEventListener("resize", updateWindowSize);

var windowSize = function () {
  return {
    width: data.get("width"),
    height: data.get("height")
  };
};
```

###用Deps.Dependency 创建reactive value

创建reactive value 的另一种方式就是使用Deps.Dependency helper 对象，它包含在Deps主库中。

假设我们有一个favoriteFood 变量和getter 以及setter函数getFavoriteFood 和setFavoriteFood:

```
var favoriteFood = "apples";

var getFavoriteFood = function () {
  return favoriteFood;
};

var setFavoriteFood = function (newValue) {
  favoriteFood = newValue;
};

`favoriteFood` can be made into a reactive variable by adding just four lines of code.

var favoriteFood = "apples";
var favoriteFoodDep = new Deps.Dependency;  // FIRST ADDED LINE

var getFavoriteFood = function () {
  favoriteFoodDep.depend();                 // SECOND ADDED LINE
  return favoriteFood;
};

var setFavoriteFood = function (newValue) {
  if (newValue !== favoriteFood)            // THIRD ADDED LINE
    favoriteFoodDep.changed();              // FOURTH ADDED LINE
  favoriteFood = newValue;
};

```
第一行创建了一个Deps.Dependency 对象，叫做favoriteFoodDep，它用来保存关注favoriteFood 变量值的autorun列表。

第二行，当favoriteFood 被读取时，当前的autorun会被添加到favoriteFoodDep 中，这是通过depend()函数来做。如果不是在autorun中调用的getFavoriteFood()，则什么也不做。

第三、四行，当favoriteFood发生变化时，所有记录在favoriteFoodDep中的autorun都被通知重新执行。


###Deps.Computation 对象

Deps.Dependency 是一个方便的辅助对象。它构建于Deps 对象，Deps.Computation 之上。

每次调用Deps.autorun 时都会创建一个Deps.Computation 对象。它代表当前autorun的实例，它含有一个stop()方法，用于终止和清理本autorun。

Computation 还有两个方法：
- invalidate(): 导致重新运行autorun本身
- onInvalidate(callback):下次autorun重新运行时或是stop时执行callback

它还包含三个属性，stopped,invalidated 和firstRun

代码通过查询全局变量Deps.currentComputation来看是否运行在autorun内部。如果在autorun外面，Deps.currentComputation 为null，如果在autorun内部，则Deps.currentComputation 就是代表当前autorun的Deps.Computation对象。当在嵌套autorun时，Deps.Computation 代表父autorun，或者说是包裹代码的最小的autorun。

一个Deps.Dependency 是一个容器，保存一系列Deps.Computation对象。depend()方法会把Deps.curentComputation添加到列表中，如果存在的话。changed()方法调用列表里每一个computation的invalidate()方法。

Deps.Dependency 还给每一个它跟踪的computation建立起一个onInvalidate 回调函数。当一个computation invalid时候，回调函数会把它重跟踪列表里移除。Computations are invalidated by a call to changed() on a Deps.Dependency, by a call to changed() on some other Deps.Dependency that is used by the same computation, or by a call to stop() on an autorun.

基于Deps.Computation API 你可以实现自己的Deps.Dependency ，例如：

```
Dependency = function () {
  this._nextId = 0;
  this._dependents = {};
};

Dependency.prototype.depend = function () {
  var self = this;
  if (Deps.currentComputation) {
    var id = self._nextId++;
    self._dependents[id] = Deps.currentComputation;
    Deps.currentComputation.onInvalidate(function () {
      delete self._dependents[id];
    });
  }
};

Dependency.prototype.changed = function () {
  for (var id in this._dependents) {
    this._dependents[id].invalidate();
  }
};
```

###创建reactive value the hard way 
###Computation 和Dependencies 多对多关系

要想完全理解Deps,知道下面代码为何错误非常关键：

```
// WRONG
Dependency = function () {
  this._nextId = 0;
  this._dependents = {};
};

Dependency.prototype.depend = function () {
  if (Deps.currentComputation) {
    var id = self._nextId++;
    this._dependents[id] = Deps.currentComputation;
  }
};

Dependency.prototype.changed = function () {
  for (var id in this._dependents) {
    this._dependents[id].invalidate();
  }
  this._dependents = {};
};
// WRONG
```



###taking  a "wait and see"approach



##8. Flush Cycle

autorun什么时候重新运行？换句话说，改变一个reactive value后何时生效？

```
var data = new ReactiveDict;
data.set("favoriteFood", "chicken");

Deps.autorun(function () {
  console.log(data.get("favoriteFood"));
});

console.log("start update");
data.set("favoriteFood", "waffles");
data.set("favoriteFood", "pie");
console.log("finish update");
```


"waffles"和"pie"会在"finish"之前还是之后打印出来。

答案是：默认情况下，当所有javascript代码都结束执行，系统处于空闲时，reactive value把所有变化批处理，实施一次(例如：在浏览器里，可能就是在浏览器处理完当前事件之后)。批处理的过程叫做flushing，可以通过调用Deps.flush()手动触发。

所以上面的代码会输出：

```
chicken
start update
finish update
waffles
pie
```

如果用Deps.flush()修改上面的例子：

```
var data = new ReactiveDict;
data.set("favoriteFood", "chicken");

Deps.autorun(function () {
  console.log(data.get("favoriteFood"));
});

console.log("start update");
data.set("favoriteFood", "waffles");
data.set("favoriteFood", "pie");
Deps.flush(); // ADDED LINE
console.log("finish update");
```

结果就是：

```
chicken
start update
waffles
pie
finish update
```

###为什么要有flush
- 可预测:最重要的原因就是可预测。不Flush保证什么都不会变，Flush保证所有更新都完成。
- 一致性：Flush使系统从一个一致的状态到另一个一致的状态。
- 性能



###hook 进Flush cycle
使用Deps.afterFlush() 来在下一次Flush cycle完成时运行回调函数（在所有autorun都运行过后）

```
var data = new ReactiveDict;
data.set("favoriteFood", "cake");

Deps.autorun(function () {
  console.log("My favorite food is " + data.get("favoriteFood") + "!");
});

var setUnpopularFood = function (what) {
  data.set("favoriteFood", what);
  Deps.afterFlush(function () {
    console.log("Sounds gross to you, but from where I'm from it's considered a delicacy!");
  });
};
```

在上面的例子中，

```
set("favoriteFood", "candy");
// "My favorite food is candy!"
setUnpopularFood("lizards");
// "My favorite food is lizards!"
// "Sounds gross to you, but from where I'm from it's considered a delicacy!"
set("favoriteFood", "ice cream");
// "My favorite food is ice cream!"
```

Deps.afterFlush 是一次性的，在下次Flush cycle之后运行一次。以后就不再运行。
###Flush cycle如何工作

每一个autorun都有一个标识，invalidate,表明是否需要重新运行。当一个reactive 值发生变化时，会在所有影响的Computation 上设为true，意味着在下一次Flush的时候他们需要重新运行。当Flush被触发时，可以是通过Deps.flush()也可以是由于系统空闲。所有invalidate的Computation 都会重新运行，它们的invalidate 标识被清除。Flush的最后，所有afterFlush handler执行。

有下面的保证：
- 当Deps.flush()返回时，所有状态都被更新。变化影响到的每一个autorun都结束了重新运行。autorun的执行顺序不确定。即使在Flush cycle中reactive value发生变化时，这一点也能保证。
- 当Deps.flush()返回时，所有的afterFLush 处理器都会运行，即使当Flush开始时，afterFlush还没注册。直到没有需要重新运行的autorun时，afterFlush 才会执行。afterFlush按照注册顺序执行。

即使在Flush cycle中reactive value发生变化，或在Flush cycle中新注册了afterFlush处理器，也能保证上面两点。这暗示：
- 有时候，重新运行一个autorun会改变一个reactive value，当发生这种情况时，依赖于这个reactive value的autorun也会在本次Flush cycle中重新运行。
- 不能再一个autorun中调用Flush() ，否则你会无限循环。Flush() 只能在所有autorun返回之后才能返回，当时autorun不能返回，直到flush() 返回。
- 如果一个afterFlush 回调函数改变了一个reactive value，依赖于该reactive value的autorun都会在任何其它afterFlush回调运行前重新运行。
- 如果一个afterFlush回调函数初测了一个新的afterFlush处理器，它会作为当前Flush cycle的一部分运行。

###避免change loop
- 规则一：除非真发生了变化，否则不要调用Deps.changed()
- 规则二：不要创建循环引用。

