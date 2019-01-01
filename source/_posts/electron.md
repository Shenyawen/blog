---
title: electron 入门
date: 2018-12-29 10:53:00
tags: electron
---
最近一直在使用electron开发桌面应用，在这里可以跟大家做下简单的分享
## 简介
electron是一款web桌面应用框架，基于NodeJs跟Chromium, 大部分项目主要分成两个进程：render进程和main进程。成本相较于C C#很低，大部分情况下在牺牲一部分性能的基础上提高开发效率还是值得的

## 优势
我认为主要有下面几点吧
1.快，那是真快。一套代码输出window mac端，而且绝大部分代码都是基于js，跟正常的web应用开发几乎没有差别，写web有多快，写它就有多快
2.因为是基于Chromium, 所以web开发中常见的兼容性问题基本没有，写起来很流畅
3.系统是运行在node环境的，node的那些api都可以使用，突破浏览器的一系列限制，可以很方便的跟本地系统交互，可以实现更多纯web做不到的事情

electron现在已经有中文官方文档了 https://electronjs.org/docs

github上也有大量的electron demo可以借鉴学习

打包脚手架有electron-vue, 可以看一下，入门还是很好的
