---
layout: post
category : lessons
title: "理解 javascript 中的对象"
tagline: "Supporting tagline"
tags : [javascript]
---

keep in mind that objects in JavaScript are dynamic, meaning that they can change at any point during code execution. Whereas class-based languages lock down objects based on a class definition, JavaScript objects have no such restrictions

由于对象是动态的，所以经常需要检测对象是否包含某属性。使用 in 不要使用 object.property 形式。The in operator checks for both own properties and prototype properties。

hasOwnProperty

delete 操作符删除属性。delete only works on own properties

for (property in object)
Object.keys()
propertyIsEnumerable

属性类型
属性分为两种：data properties and accessor properties.
Accessor properties are most useful when you want the assignment of a value to trigger some sort of behavior, or when reading a value requires the calculation of the desired return value.

属性特性：

    - [[Enumerable]]
    - [[Configurable]]
    - [[Value]]
    - [[Writable]]
    - [[Get]] and [[Set]]

Object .defineProperty() 修改属性特性
Object.defineProperties()
Object.getOwnPropertyDescriptor()

防止对象被修改
三种方式：

- Preventing Extensions
    - 对象和属性一样，也具有特性。
[[Extensible]]
Object.preventExtensions()
- Sealing Objects
    - A sealed object is nonextensible, and all of its properties are nonconfigu-rable. That means not only can you not add new properties to the object, but you also can’t remove properties or change their type (from data to accessor or vice versa). If an object is sealed, you can only read from and write to its properties
    - Object.seal()
    - Ineffect, sealed objects are JavaScript’s way ofgiving you the same mea-sure of control without using classes.
- Freezing Objects
    - a frozen object is a sealed object where data properties are also read-only. Frozen objects can’t become  unfrozen
    - Object.freeze()
    - Object.isFrozen()
