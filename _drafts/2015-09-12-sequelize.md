---
layout: post
category : lessons
title: "sequelize 学习"
tagline: "Supporting tagline"
tags : [sequelize]
---

## 开始

### 安装

```
$ npm install --save sequelize

# 安装下面的一种
$ npm install --save pg pg-hstore
$ npm install --save mysql // For both mysql and mariadb dialects
$ npm install --save sqlite3
$ npm install --save tedious // MSSQL
```

### 建立连接

```
var sequelize = new Sequelize('database', 'username', 'password', {
  host: 'localhost',
  dialect: 'mysql'|'mariadb'|'sqlite'|'postgres'|'mssql',

  pool: {
    max: 5,
    min: 0,
    idle: 10000
  },

  // SQLite only
  storage: 'path/to/database.sqlite'
});

// 或者提供一个连接地址
var sequelize = new Sequelize('postgres://user:pass@example.com:5432/dbname');
```

### 定义模型

模型通过 `sequelize.define('name', {attributes}, {options})` 定义

## 表结构
`sequelize.sync()`会基于模型定义，创建缺失的表。如果提供了 `force:true` 在重建表之前会先drop 掉。

## 模型

### 定义
使用 `define` 方法定义模型和数据表之间的映射。Sequelize会自动添加属性 createAt 和 updateAt

#### 数据类型

#### getter 和 setter

#### 校验

#### 配置

#### 导入

#### 数据库同步

#### 扩展模型

#### 索引

#### 使用

查找指定记录。findById() 或是 findOne()

```
/ search for known ids
Project.findById(123).then(function(project) {
  // project will be an instance of Project and stores the content of the table entry
  // with id 123. if such an entry is not defined you will get null
})

// search for attributes
Project.findOne({ where: {title: 'aProject'} }).then(function(project) {
  // project will be the first entry of the Projects table with the title 'aProject' || null
})


Project.findOne({
  where: {title: 'aProject'},
  attributes: ['id', ['name', 'title']]
}).then(function(project) {
  // project will be the first entry of the Projects table with the title 'aProject' || null
  // project.title will contain the name of the project
})
```

查找指定记录，不存在的话就创建。 findOrCreate()

```

```

查找多个记录，返回记录和总数。

```
Project
  .findAndCountAll({
     where: {
        title: {
          $like: 'foo%'
        }
     },
     offset: 10,
     limit: 2
  })
  .then(function(result) {
    console.log(result.count);
    console.log(result.rows);
  });
```

查找多条记录

```
// find multiple entries
Project.findAll().then(function(projects) {
  // projects will be an array of all Project instances
})

// also possible:
Project.all().then(function(projects) {
  // projects will be an array of all Project instances
})

// search for specific attributes - hash usage
Project.findAll({ where: { name: 'A Project' } }).then(function(projects) {
  // projects will be an array of Project instances with the specified name
})

// search with string replacements
Project.findAll({ where: ["id > ?", 25] }).then(function(projects) {
  // projects will be an array of Projects having a greater id than 25
})

// search within a specific range
Project.findAll({ where: { id: [1,2,3] } }).then(function(projects) {
  // projects will be an array of Projects having the id 1, 2 or 3
  // this is actually doing an IN query
})

Project.findAll({
  where: {
    id: {
      $and: {a: 5}           // AND (a = 5)
      $or: [{a: 5}, {a: 6}]  // (a = 5 OR a = 6)
      $gt: 6,                // id > 6
      $gte: 6,               // id >= 6
      $lt: 10,               // id < 10
      $lte: 10,              // id <= 10
      $ne: 20,               // id != 20
      $between: [6, 10],     // BETWEEN 6 AND 10
      $notBetween: [11, 15], // NOT BETWEEN 11 AND 15
      $in: [1, 2],           // IN [1, 2]
      $notIn: [1, 2],        // NOT IN [1, 2]
      $like: '%hat',         // LIKE '%hat'
      $notLike: '%hat'       // NOT LIKE '%hat'
      $iLike: '%hat'         // ILIKE '%hat' (case insensitive)
      $notILike: '%hat'      // NOT ILIKE '%hat'
      $overlap: [1, 2]       // && [1, 2] (PG array overlap operator)
      $contains: [1, 2]      // @> [1, 2] (PG array contains operator)
      $contained: [1, 2]     // <@ [1, 2] (PG array contained by operator)
      $any: [2,3]            // ANY ARRAY[2, 3]::INTEGER (PG only)
    },
    status: {
      $not: false,           // status NOT FALSE
    }
  }
})
```

复杂过滤 $and $or $not

用 limit ,offset ,order , 和group操作数据集

```
/ limit the results of the query
Project.findAll({ limit: 10 })

// step over the first 10 elements
Project.findAll({ offset: 10 })

// step over the first 10 elements, and take 2
Project.findAll({ offset: 10, limit: 2 })
```

原生查询
有时候需要查询大量数据，只是为了展示，无需操作。

```
// Are you expecting a massive dataset from the DB,
// and don't want to spend the time building DAOs for each entry?
// You can pass an extra query option to get the raw data instead:
Project.findAll({ where: { ... }, raw: true })
```

计数

```
Project.count().then(function(c) {
  console.log("There are " + c + " projects!")
})

Project.count({ where: ["id > ?", 25] }).then(function(c) {
  console.log("There are " + c + " projects with an id greater than 25.")
})
```

获取最大值

获取最小值

求和

## 查询
### 属性
只查询特定属性：

```
Model.findAll({
  attributes: ['foo', 'bar']
});
```

重命名属性：用嵌套的数组

```
Model.findAll({
  attributes: ['foo', ['bar', 'baz']]
});
```

等价于：

```
SELECT foo, bar AS baz ...
```

使用 sequelize.fn 来做聚合：

```
Model.findAll({
  attributes: [sequelize.fn('COUNT', sequelize.col('hats')), 'no_hats']
});
```

等价于：
```
SELECT COUNT(hats) AS no_hats ...
```

使用聚合函数时必须提供别名，这样才能通过模型获取。

可以使用 include 和 exclude 来包含或排除一些属性。

### where

### JSONB

### 关联

### 分页

### 排序



## 实例

### 构建非持久实例
build 方法会返回一个未保存的对象。构建实例会自动获得定义时的默认值。

```
var task = Task.build({
  title: 'specify the project idea',
  description: 'bla',
  deadline: new Date()
})
```

保存到数据库使用 save() 方法。

```
project.save().then(function() {
  // my nice callback stuff
})
 
task.save().catch(function(error) {
  // mhhh, wth!
})
 
// you can also build, save and access the object with chaining:
Task
  .build({ title: 'foo', description: 'bar', deadline: new Date() })
  .save()
  .then(function(anotherTask) {
    // you can now access the currently saved task with the variable anotherTask... nice!
  }).catch(function(error) {
    // Ooops, do some error-handling
  })
```

### 构建持久实例

直接一步保存到数据库。 create 方法

```
Task.create({ title: 'foo', description: 'bar', deadline: new Date() }).then(function(task) {
  // you can now access the newly created task via the variable task
})
```

可以通过fields 来限制哪些属性可以保存到数据库

```
User.create({ username: 'barfooz', isAdmin: true }, { fields: [ 'username' ] }).then(function(user) {
  // let's assume the default of isAdmin is false:
  console.log(user.get({
    plain: true
  })) // => { username: 'barfooz', isAdmin: false }
})
```

### 更新、保存、持久化一个实例

```
// way 1
task.title = 'a very different title now'
task.save().then(function() {})
 
// way 2
task.update({
  title: 'a very different title now'
}).then(function() {})
```

也可以通过fields 来限制哪些属性可以保存到数据库

### 销毁、删除 持久化的实例

```
Task.create({ title: 'a task' }).then(function(task) {
  // now you see me...
 
  task.destroy().then(function() {
    // now i'm gone :)
  })
})
```

如果 paranoid 选项为true，对象并不会被真删除，deleteAt 列会被设置为当前时间戳。要强制删除的话，可以传入 force:true 

```
task.destroy({force:true})
```

### 批量增删改


## 当表已经存在时

```
sequelize.define('User', {

}, {
  tableName: 'users'
});
```

## 关联
当我们调用类似 User.hasOne(Project) 的方法时，我们把User称为源，Project称为目标。

### 一对一
一对一关系是两个模型通过外键关联

#### BelongsTO
外键存在于源。

#### 外键
在belongTo 关系中，默认情况下，外键由目标模型和目标主键名生成。默认是驼峰形式，如果源模型配置了 `underscored: true` ，外键将是 `snake_case ` 形式。

可以通过 foreignKey 选项 重写默认的外键。

#### 目标键
在belongTo 关系中，默认的目标键是目标模型的逐渐。可以用 targetKey 选项重写。

#### HasOne
外键存在于目标模型。

### 一对多
一个源对应多个目标。一个目标只对应一个源。

### 属于多个
一个源对应多个目标。一个目标对应多个源。

会创建一个中间模型。

### scope 
### 命名策略
### 关联对象

