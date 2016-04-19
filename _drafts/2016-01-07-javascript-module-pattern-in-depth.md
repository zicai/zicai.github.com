---
layout: post
category : lessons
title: "【译】深入解读 JavaScript module pattern"
tagline: "Supporting tagline"
tags : [javascript]
---

## 基础

### Anonymous Closures

```
(function () {
	// ... all vars and functions are in this scope only
	// still maintains access to all globals
}());
```

### Global Import

```
(function ($, YAHOO) {
	// now have access to globals jQuery (as $) and YAHOO in this code
}(jQuery, YAHOO));
```

### Module Export

```
var MODULE = (function () {
	var my = {},
		privateVariable = 1;

	function privateMethod() {
		// ...
	}

	my.moduleProperty = 1;
	my.moduleMethod = function () {
		// ...
	};

	return my;
}());
```




## 高级
### Augmentation
### Loose Augmentation
### Tight Augmentation

### Cloning and Inheritance
### Cross-File Private State
### Sub-modules


## 结论

原文地址：

- http://www.adequatelygood.com/JavaScript-Module-Pattern-In-Depth.html


















