---
title: JavaScript 模块化
date: 2019-09-30 14:06:04
tags: 面试
categories: [JavaScript]
---

 随着前端js代码复杂度的提高，js模块化是必然趋势，不仅好维护，同时依赖很明确，不会全局污染，在这里整理一下对JavaScript各种模块的理解。

<!-- more -->

### 为什么要使用模块化

使用一个技术肯定是有原因的，那么使用模块化可以给我们带来以下好处

- 解决命名冲突
- 提供复用性
- 提高代码可维护性

### Common.js

> 参考链接: [阮一峰 common.js 规范](https://javascript.ruanyifeng.com/nodejs/module.html)

Node 应用由模块组成，采用 CommonJS 模块规范。

每个文件就是一个模块，有自己的作用域。在一个文件里面定义的变量、函数、类，都是私有的，对其他文件不可见。CommonJS规范规定，每个模块内部，`module`变量代表当前模块。这个变量是一个对象，它的`exports`属性（即`module.exports`）是对外的接口。加载某个模块，其实是加载该模块的`module.exports`属性。

* module对象

Node内部提供一个`Module`构建函数。所有模块都是`Module`的实例。

```javascript
function Module(id, parent) {
  this.id = id;
  this.exports = {};
  this.parent = parent;
  //...
 }
```

每一个模块内部，都有一个`module`对象，代表当前模块它有以下属性

```
module.id 模块的识别符，通常是带有绝对路径的模块文件名。
module.filename 模块的文件名，带有绝对路径。
module.loaded 返回一个布尔值，表示模块是否已经完成加载。
module.parent 返回一个对象，表示调用该模块的模块。
module.children 返回一个数组，表示该模块要用到的其他模块。
module.exports 表示模块对外输出的值。
```

* module.exports/export对象

`module.exports`属性表示当前模块对外输出的接口

```javascript
var EventEmitter = require('events').EventEmitter;
module.exports = new EventEmitter();

setTimeout(function() {
  module.exports.emit('ready');
}, 1000);
```

上面模块会在加载后1秒后，发出ready事件。其他文件监听该事件，可以写成下面这样。

```javascript
var a = require('./a');
a.on('ready', function() {
  console.log('module a is ready');
});
```

`exports = module.exports`

因此，我们可以用下列的方式对`exports`赋值

```javascript
exports.area = function (r) {
  return Math.PI * r * r;
};

exports.circumference = function (r) {
  return 2 * Math.PI * r;
};
```

但是不能对 `exports` 直接赋值。因为 `var exports = module.exports` 这句代码表明了 `exports` 和 `module.exports` 享有相同地址，通过改变对象的属性值会对两者都起效，但是如果直接对 `exports` 赋值就会导致两者不再指向同一个内存地址，修改并不会对 `module.exports` 起效。

