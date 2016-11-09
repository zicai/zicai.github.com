---
layout: post
category : 数据库
title: "SQL 基础知识"
tagline: "Supporting tagline"
tags : [sql]
---



可以把 SQL 分为两个部分：

- 数据操作语言 (DML) 
- 数据定义语言 (DDL)


SQL (结构化查询语言)是用于执行查询的语法。但是 SQL 语言也包含用于更新、插入和删除记录的语法。查询和更新指令构成了 SQL 的 DML 部分：

- SELECT - 从数据库表中获取数据
- UPDATE - 更新数据库表中的数据
- DELETE - 从数据库表中删除数据
- INSERT INTO - 向数据库表中插入数据

SQL 的数据定义语言 (DDL) 部分使我们有能力创建或删除表格。我们也可以定义索引（键），规定表之间的链接，以及施加表间的约束。SQL 中最重要的 DDL 语句:

- CREATE DATABASE - 创建新数据库
- ALTER DATABASE - 修改数据库
- CREATE TABLE - 创建新表
- ALTER TABLE - 变更（改变）数据库表
- DROP TABLE - 删除表
- CREATE INDEX - 创建索引（搜索键）
- DROP INDEX - 删除索引

注意：

- SQL 对大小写不敏感！
- SQL 使用单引号来环绕文本值（大部分数据库系统也接受双引号）。如果是数值，请不要使用引号。

## select

```
SELECT 列名称 FROM 表名称
```
 
列出不同值

```
SELECT DISTINCT 列名称 FROM 表名称
```

## where 子句

```
SELECT 列名称 FROM 表名称 WHERE 列 运算符 值
```

### 运算符
运算符包括：等于、不等于（<>,!=）、大于、小于、大于等于、小于等于、在某个范围内（between）、搜索某种模式（like）

#### AND & OR 运算符
AND 和 OR 可在 WHERE 子语句中把两个或多个条件结合起来。也可以把 AND 和 OR 结合起来（使用圆括号来组成复杂的表达式）

#### like
LIKE 操作符用于在 WHERE 子句中搜索列中的指定模式。

```
SELECT column_name(s) FROM table_name WHERE column_name LIKE pattern
```

"%" 可用于定义通配符（模式中缺少的字母）

使用 NOT 关键字:     not like pattern

#### 通配符
SQL 通配符必须与 LIKE 运算符一起使用。

| 通配符 | 含义 |
|:---:|:---:|
| % | 替代一个或多个字符 |
| _ | 仅替代一个字符|
| [charlist] |字符列中的任何单一字符|
| [^charlist] 或者 [!charlist] | 不在字符列中的任何单一字符|


### Between
BETWEEN 操作符在 WHERE 子句中使用，作用是选取介于两个值之间的数据范围。这些值可以是数值、文本或者日期。

```
SELECT column_name(s)
FROM table_name
WHERE column_name
BETWEEN value1 AND value2
```
也可以使用 `not between value1 AND value2`

### IN
IN 操作符允许我们在 WHERE 子句中规定多个值。

```
SELECT column_name(s)
FROM table_name
WHERE column_name IN (value1,value2,...)
```


## Group BY

合计函数 (比如 SUM) 常常需要添加 GROUP BY 语句。



## Having

在 SQL 中增加 HAVING 子句原因是，WHERE 关键字无法与合计函数一起使用。

例如：

```
SELECT column_name, aggregate_function(column_name)
FROM table_name
WHERE column_name operator value
GROUP BY column_name
HAVING aggregate_function(column_name) operator value
```




## Order by

```
order by 列名称 ,列名称 DESC
```

ORDER BY 语句默认按照升序对记录进行排序。如果您希望按照降序对记录进行排序，可以使用 DESC 关键字。

## INSERT INTO

```
INSERT INTO table_name (列1, 列2,...) VALUES (值1, 值2,....)
```

## update

```
UPDATE 表名称 SET 列名称 = 新值 WHERE 列名称 = 某值
```

## delete

```
DELETE FROM 表名称 WHERE 列名称 = 值

// 删除所有行
DELETE * FROM table_name 
```
## Top
TOP 子句用于规定要返回的记录的数目。

```
sqlserver，Oracle :     
SELECT TOP number|percent column_name(s) FROM table_name

mysql:                         
SELECT column_name(s) FROM table_name LIMIT number

Oracle                         
SELECT column_name(s) FROM table_name WHERE ROWNUM <= number

// 选取百分比
SELECT TOP 50 PERCENT * FROM Persons
```


## 别名
通过使用 SQL，可以为列名称和表名称指定别名（Alias）。

给表指定别名：

```
SELECT column_name(s)
FROM table_name
AS alias_name
```
例如：

```
SELECT po.OrderID, p.LastName, p.FirstName
FROM Persons AS p, Product_Orders AS po
WHERE p.LastName='Adams' AND p.FirstName='John'
```

给列指定别名：（改变结果集的列名）

```
SELECT column_name AS alias_name
FROM table_name
```
## 连表查询

1. 可以通过引用两个表的方式，从两个表中获取数据

	```
SELECT Persons.LastName, Persons.FirstName, Orders.OrderNo
FROM Persons, Orders
WHERE Persons.Id_P = Orders.Id_P
```

2. 也可以使用关键词 JOIN 来从两个表中获取数据。

下面列出了您可以使用的 JOIN 类型，以及它们之间的差异:

- INNER JOIN: 只返回匹配行
- LEFT JOIN: 即使右表中没有匹配，也从左表返回所有的行
- RIGHT JOIN: 即使左表中没有匹配，也从右表返回所有的行
- FULL JOIN: 从左表 (Persons) 和右表 (Orders) 那里返回所有的行

## UNION
UNION 操作符用于合并两个或多个 SELECT 语句的结果集。

请注意，UNION 内部的 SELECT 语句必须拥有相同数量的列。列也必须拥有相似的数据类型。同时，每条 SELECT 语句中的列的顺序必须相同。

```
SELECT column_name(s) FROM table_name1
UNION
SELECT column_name(s) FROM table_name2
```

默认地，UNION 操作符选取不同的值。如果允许重复的值，请使用 UNION ALL。
UNION 结果集中的列名总是等于 UNION 中第一个 SELECT 语句中的列名。

## SELECT INTO
SELECT INTO 语句可用于创建表的备份。从一个表中选取数据，然后把数据插入另一个表中。
可以带 where 子句，可以连表查

```
SELECT *
INTO new_table_name [IN externaldatabase]
FROM old_tablename

或
SELECT column_name(s)
INTO new_table_name [IN externaldatabase]
FROM old_tablename
```
## 创建数据库
CREATE DATABASE database_name

## 创建表

```
CREATE TABLE 表名称
(
列名称1 数据类型,
列名称2 数据类型,
列名称3 数据类型,
....
)
```

## 数据类型
| 类型 | 说明 |
|:---:|:---:|
| integer(size) <br> int(size) <br> smallint(size) <br> tinyint(size)| 仅容纳整数。在括号内规定数字的最大位数。|
|decimal(size,d) <br> numeric(size,d)|容纳带有小数的数字。"size" 规定数字的最大位数。"d" 规定小数点右侧的最大位数。|
| char(size) |容纳固定长度的字符串（可容纳字母、数字以及特殊字符）。在括号中规定字符串的长度。|
| varchar(size) | 容纳可变长度的字符串（可容纳字母、数字以及特殊的字符）。在括号中规定字符串的最大长度。|
| date(yyyymmdd) | 容纳日期。|

所有数据类型 http://www.w3school.com.cn/sql/sql_datatypes.asp

## 约束
可以在创建表时规定约束（通过 CREATE TABLE 语句），或者在表创建之后也可以（通过 ALTER TABLE 语句）
不同的数据库语法不同。

### NOT NULL
NOT NULL 约束强制字段始终包含值。这意味着，如果不向字段添加值，就无法插入新记录或者更新记录。

### UNIQUE
UNIQUE 和 PRIMARY KEY 约束均为列或列集合提供了唯一性的保证。
PRIMARY KEY 拥有自动定义的 UNIQUE 约束。
请注意，每个表可以有多个 UNIQUE 约束，但是每个表只能有一个 PRIMARY KEY 约束。

创建表时：

```
MySQL:
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255)
UNIQUE (Id_P)
)

SQL Server / Oracle / MS Access:
CREATE TABLE Persons
(
Id_P int NOT NULL UNIQUE,
LastName varchar(255) NOT NULL,
FirstName varchar(255)
)
```

如果需要命名 UNIQUE 约束，以及为多个列定义 UNIQUE 约束，请使用下面的 SQL 语法

```
MySQL / SQL Server / Oracle / MS Access:
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
CONSTRAINT uc_PersonID UNIQUE (Id_P,LastName)
)
```

修改表

```
MySQL / SQL Server / Oracle / MS Access:
ALTER TABLE Persons
ADD UNIQUE (Id_P)
```

如需命名 UNIQUE 约束，并定义多个列的 UNIQUE 约束，请使用下面的 SQL 语法：

```
MySQL / SQL Server / Oracle / MS Access:
ALTER TABLE Persons
ADD CONSTRAINT uc_PersonID UNIQUE (Id_P,LastName)
```
撤销约束

```
MySQL:
ALTER TABLE Persons
DROP INDEX uc_PersonID

SQL Server / Oracle / MS Access:
ALTER TABLE Persons
DROP CONSTRAINT uc_PersonID
```

### PRIMARY KEY

建表时:

```
MySQL:
CREATE TABLE Persons
(
Id_P int NOT NULL,
PRIMARY KEY (Id_P)
)

SQL Server / Oracle / MS Access:
CREATE TABLE Persons
(
Id_P int NOT NULL PRIMARY KEY
)
```

如果需要命名 PRIMARY KEY 约束，以及为多个列定义 PRIMARY KEY 约束，请使用下面的 SQL 语法：

```
MySQL / SQL Server / Oracle / MS Access:
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL
CONSTRAINT pk_PersonID PRIMARY KEY (Id_P,LastName)
)
```

修改表
```
MySQL / SQL Server / Oracle / MS Access:
ALTER TABLE Persons
ADD PRIMARY KEY (Id_P)
```

如果需要命名 PRIMARY KEY 约束，以及为多个列定义 PRIMARY KEY 约束，请使用下面的 SQL 语法：

```
MySQL / SQL Server / Oracle / MS Access:
ALTER TABLE Persons
ADD CONSTRAINT pk_PersonID PRIMARY KEY (Id_P,LastName)

```
注释：如果您使用 ALTER TABLE 语句添加主键，必须把主键列声明为不包含 NULL 值（在表首次创建时）。

撤销主键

```
MySQL:
ALTER TABLE Persons
DROP PRIMARY KEY

SQL Server / Oracle / MS Access:
ALTER TABLE Persons
DROP CONSTRAINT pk_PersonID
```

### FOREIGN KEY
一个表中的 FOREIGN KEY 指向另一个表中的 PRIMARY KEY。

### CHECK
CHECK 约束用于限制列中的值的范围。

### DEFAULT
DEFAULT 约束用于向列中插入默认值

## 索引

## DROP
使用 DROP 语句，可以轻松地删除索引、表和数据库

仅仅需要除去表内的数据，但并不删除表本身
TRUNCATE TABLE 表名称

## ALTER
ALTER TABLE 语句用于在已有的表中添加、修改或删除列。

## Auto-increment
会在新记录插入表中时生成一个唯一的数字。

## 视图
视图是可视化的表

## SQL DATE 函数

## SQL NULL 值
 IS NULL 和 IS NOT NULL 操作符
SQL ISNULL()、NVL()、IFNULL() 和 COALESCE() 函数

## 函数

函数的语法：

```
SELECT function(列) FROM 表
```

函数的基本类型有：

- Aggregate 函数：操作一系列值，返回一个单一值
- Scalar 函数：操作某个单一值，返回一个单一值

### Aggregate 函数

- avg
- count
	- count(column) 返回某列的行数（不包括 NULL 值）
	- count(*) 返回被选行数
- first
- last
- max
- min
- sum
- 等

注意：如果在 SELECT 语句的项目列表中的众多其它表达式中使用 SELECT 语句，则这个 SELECT 必须使用 GROUP BY 语句！

### Scalar 函数

- ucase
- now
- mod
- 等
