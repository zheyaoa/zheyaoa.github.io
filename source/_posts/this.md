---
title: JavaScript 普通函数和箭头函数引发的一些思考 
date: 2019-07-23 14:24:30
tags: this
categories: JavaScript
---

最近碰到了一个很基础的问题,看似很基础,但是却与我对整个JavaScript的理解不大一致。所以花了很长一段事件去弄懂它(还是JavaScript基础不够扎实啊),在这里做一个总结。

<!-- more -->

### 箭头函数和普通函数的this指向

在　JavaScript  中普通函数和箭头函数的 this 的已经是一个老生常谈的话题，相信对于任何一个前端er来说都有有一些自己的理解。我的理解是这样的

* 普通函数的 this 在函数调用时确认

  举一个见的例子把

  ```javascript
  var a = 1
  var obj = {
      a:2,
      sayA:function(){
      	console.log(this.a)   
      }
  }
  obj.sayA()   //2
  var say = obj.sayA
  say()   //1
  ```

  上面这个代码很好理解，当执行`obj.sayA()`时，函数是obj的一部分，所以此时的this 指向obj　输出为２,

  当`say = obj.say` 时，say 是window 下的一个函数，所以`say()`的结果是１

* 箭头函数的 this 在定义时确认(***this 是最近的父级上下文***)

  同样以上面这个例子为例

  ```javascript
  var a = 1
  var obj = {
      a:2,
      say()=>{
      	console.log(this.a)   
      }
  }
  obj.sayA()   //1
  var say = obj.sayA
  say()   //1
  ```

  两个函数执行都会输出1,原因很简单。使用箭头函数时的 this 在定义时就确认了。

  我们用babel转义一下上面的代码　

  ***下面是babel转码结果***

  ```javascript
  "use strict";
  var _this = void 0;
  var a = 1;
  var obj = {
    a: 2,
    say: function say() {
      console.log(_this.a);
    }
  };
  obj.sayA(); //1
  var say = obj.sayA;
  say(); //1
  ```

  可以看到this 指向了全局 window,且定义时就被确认。所以我们能输出两个1

### 引发的一些思考

* 箭头函数的  `this`为什么指向 `window`

说了那么多，看起来都非常的理所当然。可是我有了一些困惑。为什么 在箭头函数中的`this` 没有指向 `obj` 而是指向 `window`呢？如果按理解的话 `this` 指向定义的地方，那 `this`不应该是指向`obj`吗 

对此,查阅了一下资料后。我得出了一个结论。　`this` 是函数的执行上下文，　当我们使用箭头函数时,其自身的`this`由定义时的祖先的上下文所确定，但是在JavaScript中，对象是没有上下文的(有上下文的只有对象和函数)。我们来分析一下整个获取`this`的过程，首先我们获取到　箭头函数的父级 `obj` ,但是`obj`是没有 上下文的 (说人话没有 `this`),所以我们只能去 `obj` 的 父级 `window` 获取 ` this`。  

* 普通函数的 `this` 为什么指向 `obj` 呢 

说了那么多 ，可能有人又会有些疑惑。不是说 对象没有上下文吗，那为什么普通 函数的`this`指向`obj`呢(对，说的 就是我自己  =. =)

对此，我也 查阅了一些 [资料](<https://www.ibm.com/developerworks/cn/web/1207_wangqf_jsthis/index.html>) ,文中有这样一段话 

![](http://cdn.zheyao.top/this.png)

在上个疑问中我们已经知道 ，函数是有自己的上下文的。所以说并不是对象有上下文导致普通函数的`this`指向对象,而是函数有执行上下文，他的值根据调用方式的不同，也会有所不同 。

所以我们 的疑问 也能被解决了，`this` 由函数的上下文所确定，当函数由`obj.sayA()`方式调用时，此时的执行上下文(this)指向`obj`,所以在 普通函数中的 `this` 指向 `obj`