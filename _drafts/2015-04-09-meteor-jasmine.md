---
layout: post
category : lessons
title: "meteor jasmine"
tagline: "Supporting tagline"
tags : [meteor,test]
---

运行jasmine 2.1测试
##安装

安装完之后，还需安装一个reporter

```
meteor add sanjo:jasmine
meteor add velocity:html-reporter
```



##使用
本地开发模式下，测试是自动运行的

##troubleshooting
每一种测试模式（除了服务端单元测试）都会在文件夹`.meteor/local/log/`中创建一个日志文件。方便你排查问题。

##Further reading
- jasmine 2.2 DOC
- sanjo:jasmine wiki


##例子

##测试模式
每一种测试模式都有不同的特征，每一种测试模式都有自己文件夹。
###服务端
####单元测试

- 单元测试服务端代码
- 测试执行环境与app隔离
- Meteor API 和所有package都被 stub
- 放到`tests/jasmine/server/unit/`文件夹或子文件夹中


####集成测试

- 在app 的副本中运行测试
- 可以通过引入包来测试包
- 测试必须包裹在`Jasmine.onTest(function () { /* YOUR TESTS */ });`
- 放到`tests/jasmine/server/integration/`文件夹或子文件夹中


###客户端
####单元测试

- 测试客户端代码
- 测试直接运行在浏览器
- 没有自动stub的对象
- 放到`tests/jasmine/client/unit/`文件夹或子文件夹
	默认是在chrome中运行。设置环境变量`JASMINE_BROWSER`可以更换浏览器，例如：
	
	```
	JASMINE_BROWSER=PhantomJS meteor [options]
	```
	
	注意：目前只能在chrome、phantomjs、firefox中运行。
	如果想用phantomjs 运行测试，需要先全局安装：
	
	```
	npm install -g phantomjs
	```
	
####集成测试

- 测试客户端代码
- 测试直接运行在浏览器，使用的是APP副本
- 没有自动stub的对象
- 放到`tests/jasmine/client/integration/`文件夹或子文夹中
	tips:当你想测试服务端与客户端通信的时候用这种模式，其它情况使用客户端单元测试模式。
	
	
##禁用测试模式
如果你不想使用某些测试模式，可以通过环境变量关闭它们：

- JASMINE_SERVER_UNIT=0
- JASMINE_SERVER_INTEGRATION=0
- JASMINE_CLIENT_UNIT=0
- JASMINE_CLIENT_INTEGRATION=0

##运行一次测试（用于CI）

```
meteor --test
```

##stubs
在`tests/jasmine`文件夹或其子文件夹中，以`-stubs.js` 或 `-stub.js`结尾的文件会被视为stubs,会先于app 代码加载。

##Mocks
###mock meteor
`velocity:meteor-stubs` 包可以mock Meteor API。用下面代码：

```
beforeEach(function () {
  MeteorStubs.install();
});

afterEach(function () {
  MeteorStubs.uninstall();
});
```

如果是服务端单元测试，会自动做这些工作。通过环境变量`JASMINE_PACKAGES_TO_INCLUDE_IN_UNIT_TESTS`可以停用指定的包。例如：

```
export JASMINE_PACKAGES_TO_INCLUDE_IN_UNIT_TESTS=dburles:factory 
```

客户端测试需要自己做这一步。
###mock objects
通过全局函数`mock`可以mock任何对象。例如：

```
beforeEach(function () {
  mock(window, 'Players');
});
```

This will mock the Players collection for each test. The original Players collection is automatically restored after each test.

####创建复杂的mocks

##WIKI
###Database Fixtures for Integration Tests
集成测试运行在隔离的实例中，数据库为空。通常需要一些数据来进行集成测试。可以通过数据库fixtures来做。

用下面的代码：

[https://gist.github.com/Sanjo/edbc0c1848de726b78d5](https://gist.github.com/Sanjo/edbc0c1848de726b78d5)

```
/* globals
   resetDatabase: true,
   loadDefaultFixtures: true,
*/

var Future = Npm.require('fibers/future');

resetDatabase = function () {
  console.log('Resetting database');

  // safety check
  if (!process.env.IS_MIRROR) {
    console.error('velocityReset is not allowed outside of a mirror. Something has gone wrong.');
    return false;
  }

  var fut = new Future();

  var collectionsRemoved = 0;
  var db = MongoInternals.defaultRemoteCollectionDriver().mongo.db;
  db.collections(function (err, collections) {

    var appCollections = _.reject(collections, function (col) {
      return col.collectionName.indexOf('velocity') === 0 ||
        col.collectionName === 'system.indexes';
    });

    _.each(appCollections, function (appCollection) {
      appCollection.remove(function (e) {
        if (e) {
          console.error('Failed removing collection', e);
          fut.return('fail: ' + e);
        }
        collectionsRemoved++;
        console.log('Removed collection');
        if (appCollections.length === collectionsRemoved) {
          console.log('Finished resetting database');
          fut['return']('success');
        }
      });
    });

  });

  return fut.wait();
};

loadDefaultFixtures = function () {
  console.log('Loading default fixtures');
  // TODO: Insert your data into the database here
  console.log('Finished loading default fixtures');
};

if (process.env.IS_MIRROR) {
  resetDatabase();
  loadDefaultFixtures();
}
```

将上面的代码拷贝到`app/tests/jasmine/server/integration/fixtures/database-fixture.js`
`-fixture.js`很重要，它会告诉velocity在加载测试之前加载。

###端到端测试
###Iron Router的集成测试
集成测试时的时候，你希望确保在测试之前页面已经加载完成。可以通过下面的helper来做。

根据你的测试模式，将下面代码保存到`tests/jasmine/client/integration/_wait_for_router_helper.js`或
`tests/jasmine/client/unit/_wait_for_router_helper.js`

```
(function (Meteor, Tracker, Router) {
  var isRouterReady = false;
  var callbacks = [];

  window.waitForRouter = function (callback) {
    if (isRouterReady) {
      callback();
    } else {
      callbacks.push(callback);
    }
  };

  Router.onAfterAction(function () {
    if (!isRouterReady && this.ready()) {
      Tracker.afterFlush(function () {
        isRouterReady = true;
        callbacks.forEach(function (callback) {
          callback();
        });
        callbacks = []
      })
    }
  });

  Router.onRerun(function () {
    isRouterReady = false;
    this.next();
  });

  Router.onStop(function () {
    isRouterReady = false;
    if (this.next) {
      this.next();
    }
  });
})(Meteor, Tracker, Router);
```

然后添加一个`beforeEach(waitForRouter);`声明到Spec。例如：

```
describe('My Spec', function () {
  beforeEach(function (done) {
    Router.go('/myPage');
    Tracker.afterFlush(done);
  });

  beforeEach(waitForRouter);

  it('should do something', function () {
    // Your test
  });
})
```

###package mock
###template testing



















