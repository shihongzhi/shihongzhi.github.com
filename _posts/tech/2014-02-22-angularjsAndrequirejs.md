---
layout:     post
title:      angularjs和requirejs的区别
category: tech
description: angularjs和requirejs，两者都有inject的功能，那它们的区别是什么呢？
tags: gitlab
---

###两个的区别

angularjs和requirejs，两者虽然都有inject的功能，但是
requirejs inject的是 class 与 function ，表现出来的是文件方面的，
而angularjs inject的是 function实例，是module， 表现出来的是app内模块之间的依赖。

因此，两者是不冲突的，是可以完美的结合使用。

当编写比较大的angularjs项目的时候，建议采用requirejs来进行依赖管理。要不然，当代码量大量增长之后，会使得项目极其难以维护。

关于这两者关系的详细介绍，可以参看此[视频](http://www.youtube.com/watch?v=4yulGISBF8w)

