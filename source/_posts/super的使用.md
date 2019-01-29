---
title: super的使用
date: 2019-01-29 20:43:33
tags: javascript
---
一直在react Component里面用super(props)， 但是并不特别清楚这个操作符的作用，所以特地看了下官方文档，感觉就是一个语法糖，可以通过super访问父类上的方法及构造函数
super关键字用于访问和调用一个对象的父对象上的函数。

## 用法
```javascript
super([arguments]); 
// 调用 父对象/父类 的构造函数(就是constructor函数)

super.functionOnParent([arguments]); 
// 调用 父对象/父类 上的方法s
```

在构造函数中使用时，super关键字将单独出现，并且必须在使用this关键字之前使用。super关键字也可以用来调用父对象上的函数。不论是静态还是非静态的函数都能调用!

