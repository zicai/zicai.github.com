---
layout: post
category : lessons
title: "【译】深入理解 meteor deps"
tagline: "Supporting tagline"
tags : [meteor]
---

原文地址：[http://manual.meteor.com/](http://manual.meteor.com/)

第一部分是关于 Deps，一个小而强大的库，使 Meteor 可以 transparent reactive programing。

第二部分是关于 Blaze，Meteor 的 reactive 模板系统。

未来还会包括 DDP(Distributed Data Protocol) 和 Unibuild(一个构建工具和打包系统)

##0. 总览
Meteor Deps 是一个非常小(~1k)但是非常强大的库，用来在 javascript 中进行 transparent reactive programing.

##1. transparent reactive programing

我们在写软件的时候，尤其是应用软件，经常会有这样的需求：监视一些值(例如：数据库记录、表格当前选中的 item，或是窗口的宽度，或是当前时间)，当它们发生变化时更新另外一些东西。

通常有几种形式来实现：

- poll and diff:周期性的获取要监视的值，如果发生变化，则执行一些操作
- Events:要监视的值发生变化时会触发一个事件。程序的其它部分(通常叫做 controller )会安排监听事件，当事件被触发时，获取当前的值并执行其它更新。
- Bindings:

另外一种形式，还没有普遍使用，但是非常适合复杂的现代应用，就是 reactive programing。一个 reactive programing 就像电子表格系统。程序员说：A3 单元格的值应该等于 B1-B6，然后乘上 C4 的值。电子表格就负责自动的 modeling 单元格之间数据流,所以当 B2 发生变化时，A3 会自动更新。

所以 reactive programing 是一种声明式编程，程序员只需说明：应该发生什么，而不像在命令式编程中需要说明：如何发生。

reactive programing 非常适合构建用户界面。

- 如果使用 poll and diff，UI 延迟会很高。
- 如果使用 events，
- 如果使用 bindings

唯一的问题？历史上，对大多数在生产环境使用的人来说，reactive programing 系统非常复杂并且运行缓慢。它要求把 reactive 表达式模型化为树的节点，每发生一个变化都不得不进行代价高的 dataflow 分析

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

上面的代码把 `favoriteFood` 变成了 reactive value。

那么什么是 “reactive value”，如何使用它呢？

```
var handle = Deps.autorun(function () {
  console.log("Your favorite food is " + getFavoriteFood());
});
// "Your favorite food is apples"

setFavoriteFood("mangoes");
// "Your favorite food is mangoes"
```

`Deps.autorun` 会立即执行传递给他的函数。

并且，

注意：并没有注册任何时间，也没有建立任何 bindings, 就发生了。autorun 简单地记录运行函数时访问到的 reactive value，当任何值发生变化时，重新运行函数。

```
setFavoriteFood("peaches");
// "Your favorite food is peaches"
setFavoriteFood("bananas");
// "Your favorite food is bananas"
handle.stop();
setFavoriteFood("cake");
// (nothing printed)
```

就像 `setTimeout` 或是 `setInterval`，调用 `Deps.autorun` 时，它返回一个 handle，可用于后面停止 autorun。

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
Autorun 可以使任何 javascript 代码块 reactive。

唯一需要了解 Deps 的是 reactive value 的提供者(例如：数据库 library)和 reactive value 的消费者(例如：Blaze，React，famo.us这样的前端库)。其余部分的代码，根本无需知道 reactive programing 系统的存在。

##3. 用 transparent reactive programing 构建用户界面

给 library 增加 Deps support 很容易。

可以使用 `ReactiveDict`。他提供一个简单地字典对象，对象的 `get(key)` 和 `set(key)` 函数都是 reactive 的。

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

实际上，你都不用写这么多代码。利用 Meteor Blaze，用下面的模板语法：

```
<template name="weather">
  The weather here is {{forecast}}!
</template>

// In app.js
Template.weather.forecast = function () {
  return forecasts.get("san-francisco");
};
```

Blaze 解析模板，给必要的元素建立起 autorun。基本的，Blaze 做的很简单，让你可以在 HTML 模板里表达用户界面。

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

用 Deps 写的 APP，代码可以分成三部分：

- reactive 数据源
- reactive 数据消费者
- 应用逻辑

##4. Deps 如何工作
一句话总结就是：Deps.autorun 记录在执行 autorun 函数时获取到的所有 reactive value，当这些值中任何一个发生变化，就会触发一个事件导致 autorun 重新运行。

更精确的说：
每次调用 `Deps.autorun` 都会创建一个 `computation` 对象。无论何时 autorun 执行代码，全局变量 `Deps.currentComputation` 就被设置为当前的 Computation。当没有 autorun 执行时，则为 `null`。

一个 reactive 数据源（例如：Reactive 的 `get()` 方法）检查是否存在 `currentComputation`,也就是说，是否在一个 autorun 中被调用。如果是，它会保持一个引用到当前的 Computation，以后当相关数据(例如：)变化时安排在 computation 上调用方法。

换句话说：Deps 简单地提供一个全局变量，作为可能发生变化的数据和需要知道数据何时发生变化的代码的汇合点。全局变量是 reactive 数据源指明当数据发生变化时通知哪一个 autorun。

接下来的部分详细的介绍该系统

##5. 监控 reactive value

###autorun 可以嵌套
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

存在嵌套 autorun 时，依赖属于最近的 autorun。也就是说：改变内层的依赖，不会重新运行外层。而改变外层的依赖时，会重新运行外层 autorun，这会销毁并重建内层的 autorun。

尽管每次外层 autorun 运行时都会重建内层 autorun。你也无需调用内层 autorun 的 `stop()` 方法。这是因为有自动 cleanup 约定。

什么时候利用嵌套 autorun 呢？像 Blaze 这种需要细粒度更新的库。用 autorun 作为reactive 的边界

###自动 cleanup 约定

如果一个库完全符合 Deps,它必须遵循约定：如果一个在 autorun 里面创建的对象，需要 stoped 或是cleaned up，当 autorun 重新运行时它应该自动自动 stopped 。为什么这是正确的约定？大概是因为当重新运行时 autorun 会重建对象
###autorun何时抛出异常
当第一次执行的时候 autorun 抛出异常，异常会被传递到调用者。创建的 `Deps.computation` 会被销毁，也不会重新执行。

如果是已经运行过一次，重新执行的时候抛出异常，会继续执行，不管后续 rerun 是否抛出异常。

###在Autorun内部获取当前的Computation
autorun 函数被调用时会带一个参数：代表该 autorun 的 `Computation` 对象

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
如果 `status` 设置为 `over` ,那么 autorun 会自己销毁。

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

`Computation` 对象的 `firstRun` 属性指明是否是第一次执行。

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

###忽略特定 reactive value的变化
通过 `Deps.nonreactive` 来忽略特定 reactive value 的变化

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



##6. 细粒度 reactivity

`ReactiveDict` : get VS equals

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

粗放更新调用 `ReactiveDict` 的 `get("favoriteFood")`，所以它依赖 `"favoriateFood" `的值，只要 `"favoriteFood"` 发生变化，autorun 就会重新运行。

细粒度更新调用 `ReactiveDict` 的 `equal("favoriteFood","pizza")`，所以它不依赖 `"favoriteFood"` 的值，而是 `"favoriteFood"` 有一个特定值 `"pizza"`.

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
inefficient 版本依赖所有 Post，efficient 版本依赖标签为 kittens 的 post

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

如果在 autorun 里调用了 `cursor.count()`，只有当 `document` 的数量发生变化时，才会 rerun。

###细粒度和粗粒度更新指导

- 不要获取用不到的数据

##7. 创建 Reactive value

###用 ReactiveDict 库创建 reactive value

```
var data = new ReactiveDict;

var getFavoriteFood = function () {
  return data.get("favoriteFood");
};

var setFavoriteFood = function (newValue) {
  data.set("favoriteFood", newValue);
};
```

限制是：`ReactiveDict` 的值不能是对象或是数组，只能是 string ，number，boolean ，Dates,null 和 undefined

`ReactiveDict` 不包含在 main deps 库中。可以通过 `meteor add reactive-dict` 来添加。

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

###用 Deps.Dependency 创建 reactive value

创建 reactive value 的另一种方式就是使用 Deps.Dependency helper 对象，它包含在 Deps 主库中。

假设我们有一个 favoriteFood 变量和 getter 以及 setter 函数 getFavoriteFood 和 setFavoriteFood:

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
第一行创建了一个 Deps.Dependency 对象，叫做 favoriteFoodDep，它用来保存关注favoriteFood 变量值的 autorun 列表。

第二行，当 favoriteFood 被读取时，当前的 autorun 会被添加到 favoriteFoodDep 中，这是通过 depend() 函数来做。如果不是在 autorun 中调用的 getFavoriteFood()，则什么也不做。

第三、四行，当 favoriteFood 发生变化时，所有记录在 favoriteFoodDep 中的 autorun 都被通知重新执行。


###Deps.Computation 对象

Deps.Dependency 是一个方便的辅助对象。它构建于 Deps 对象，Deps.Computation 之上。

每次调用 Deps.autorun 时都会创建一个 Deps.Computation 对象。它代表当前 autorun 的实例，它含有一个 stop()方法，用于终止和清理本 autorun。

Computation 还有两个方法：
- invalidate(): 导致重新运行 autorun 本身
- onInvalidate(callback):下次 autorun 重新运行时或是 stop 时执行 callback

它还包含三个属性，stopped,invalidated 和 firstRun

代码通过查询全局变量 Deps.currentComputation 来看是否运行在 autorun 内部。如果在autorun 外面，Deps.currentComputation 为 null，如果在 autorun内部，则Deps.currentComputation 就是代表当前 autorun 的 Deps.Computation 对象。当在嵌套 autorun 时，Deps.Computation 代表父 autorun，或者说是包裹代码的最小的 autorun。

一个 Deps.Dependency 是一个容器，保存一系列 Deps.Computation 对象。depend() 方法会把 Deps.curentComputation 添加到列表中，如果存在的话。changed() 方法调用列表里每一个 computation的invalidate() 方法。

Deps.Dependency 还给每一个它跟踪的 computation 建立起一个 onInvalidate 回调函数。当一个 computation invalid 时候，回调函数会把它重跟踪列表里移除。Computations are invalidated by a call to changed() on a Deps.Dependency, by a call to changed() on some other Deps.Dependency that is used by the same computation, or by a call to stop() on an autorun.

基于 Deps.Computation API 你可以实现自己的 Deps.Dependency ，例如：

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

###创建 reactive value the hard way 
###Computation 和 Dependencies 多对多关系

要想完全理解 Deps，知道下面代码为何错误非常关键：

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

autorun 什么时候重新运行？换句话说，改变一个 reactive value 后何时生效？

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


"waffles" 和 "pie" 会在 "finish" 之前还是之后打印出来。

答案是：默认情况下，当所有 javascript 代码都结束执行，系统处于空闲时，reactive value 把所有变化批处理，实施一次(例如：在浏览器里，可能就是在浏览器处理完当前事件之后)。批处理的过程叫做 flushing，可以通过调用 `Deps.flush()` 手动触发。

所以上面的代码会输出：

```
chicken
start update
finish update
waffles
pie
```

如果用 `Deps.flush()` 修改上面的例子：

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

- 可预测:最重要的原因就是可预测。不 Flush 保证什么都不会变，Flush 保证所有更新都完成。
- 一致性：Flush 使系统从一个一致的状态到另一个一致的状态。
- 性能



###hook 进Flush cycle
使用 `Deps.afterFlush()` 来在下一次 Flush cycle 完成时运行回调函数（在所有 autorun 都运行过后）

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

Deps.afterFlush 是一次性的，在下次 Flush cycle 之后运行一次。以后就不再运行。
###Flush cycle 如何工作

每一个 autorun 都有一个标识，invalidate,表明是否需要重新运行。当一个 reactive 值发生变化时，会在所有影响的 Computation 上设为 true，意味着在下一次 Flush 的时候他们需要重新运行。当 Flush 被触发时，可以是通过 `Deps.flush()` 也可以是由于系统空闲。所有invalidate 的 Computation 都会重新运行，它们的 invalidate 标识被清除。Flush 的最后，所有 afterFlush handler 执行。

有下面的保证：

- 当 `Deps.flush()` 返回时，所有状态都被更新。变化影响到的每一个 autorun 都结束了重新运行。autorun 的执行顺序不确定。即使在 Flush cycle 中 reactive value 发生变化时，这一点也能保证。
- 当 `Deps.flush()` 返回时，所有的 afterFLush 处理器都会运行，即使当 Flush 开始时，afterFlush 还没注册。直到没有需要重新运行的 autorun 时，afterFlush 才会执行。afterFlush按照注册顺序执行。

即使在 Flush cycle 中 reactive value 发生变化，或在 Flush cycle 中新注册了afterFlush 处理器，也能保证上面两点。这暗示：

- 有时候，重新运行一个 autorun 会改变一个 reactive value，当发生这种情况时，依赖于这个 reactive value 的 autorun 也会在本次 Flush cycle 中重新运行。
- 不能再一个 autorun 中调用 `Flush()` ，否则你会无限循环。`Flush()` 只能在所有autorun 返回之后才能返回，当时autorun不能返回，直到flush() 返回。
- 如果一个 afterFlush 回调函数改变了一个 reactive value，依赖于该 reactive value的 autorun 都会在任何其它 afterFlush 回调运行前重新运行。
- 如果一个 afterFlush 回调函数初测了一个新的 afterFlush 处理器，它会作为当前 Flush cycle 的一部分运行。

###避免 change loop
- 规则一：除非真发生了变化，否则不要调用 `Deps.changed()`
- 规则二：不要创建循环引用。

