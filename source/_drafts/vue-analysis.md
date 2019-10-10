---
title: Vue.js源码分析系列（一）--响应式原理
date: 2019-07-28 15:25:08
tags:
---

* Dep 发布者
* Sub（Watcher） 订阅者

在  Vue.js  中诸如 prop/data 等对象中每一个 key 都会拥有一个自己的 Dep 对象

* 初始化 data

initState => initData

![image-20190728161302850](/Users/yuxin.yu/Library/Application Support/typora-user-images/image-20190728161302850.png)

* 初始化 prop

initState => initProps

![image-20190728161502201](/Users/yuxin.yu/Library/Application Support/typora-user-images/image-20190728161502201.png)

在defineReactive中