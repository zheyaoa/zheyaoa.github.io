---
title: ES6下的Function.bind方法
date: 2019-06-03 16:09:26
tags: prototype
categories: [JavaScript,ECMAScript6]
---

在JavaScript的使用中，**this**的指向问题始终是一个难点。不同的调用方式，会使**this**指向不同的对象。而使用**call,apply,bind**等方式，可改变this的指向,完成一些令人惊叹的黑魔法

最近了解了一下Function对象下的bind方法,同时对JavaScript对象下**this**指向,**call,apply**等方法有了更深刻的了解

<!-- more -->

### function.apply(thisArg,[argsArray])

>thisArg:　function函数运行时的this值
>
>argsArray:　一个数组或者类数组对象,其中的数组元素将作为单独的参数传给function函数 
>
>returns:  调用this值和和参数产生的结果

下面讲述一些apply的简单使用

* 用apply 将数组push 进另一个数组

```javascript
var elements = [1,2,3]
var arr = ['a'];
Array.prototype.push.apply(arr,elements)
console.log(arr,elements)
```

* 找出数组中最大/最小的数

```javascript
var nums = [1,2,3,4,5]
var max = Math.max.apply(null,nums)
var min = Math.min.apply(null,nums)
console.log(max,min)
```

### function.call(thisArg,arg1,args2,args3)

> thisArg:　function函数运行时的this值
>
> arg1, arg2, ...:　 指定的参数列表。
>
> returns:  调用this值和和参数产生的结果

`function.call` 与　`function.apply`　的本质其实是一样的。只有一个区别，就是 `call()` 方法接受的是**一个参数列表**，而 `apply()` 方法接受的是**一个包含多个参数的数组**

### function.bind(thisArg,arg1,arg2,...)

>thisArg: function函数运行时指向的this值
>
>arg1,arg2,...: 当目标函数被调用时，预先添加到绑定函数的参数列表中的参数。
>
>returns:       返回一个原函数的拷贝，并拥有指定的**this**值和初始参数。

* 创建绑定函数

`bind()`的另一个最简单的用法是使一个函数拥有预设的初始参数。只要将这些参数（如果有的话）作为`bind()`的参数写在`this`后面。当绑定函数被调用时，这些参数会被插入到目标函数的参数列表的开始位置，传递给绑定函数的参数会跟在它们后面。

```javascript
this.x = 9;    // 在浏览器中，this指向全局的 "window" 对象
var module = {
  x: 81,
  getX: function() { return this.x; }
};

module.getX(); // 81

var retrieveX = module.getX;
retrieveX();   
// 返回9 - 因为函数是在全局作用域中调用的

// 创建一个新函数，把 'this' 绑定到 module 对象
// 新手可能会将全局变量 x 与 module 的属性 x 混淆
var boundGetX = retrieveX.bind(module);
boundGetX(); // 81

```

### bind函数的初步实现

bind的实质,其实为call的语法糖。我们只需返回一个利用**Function.call**改变this指向的函数即可

```javascript
Function.prototype.bindDemo = function(obj,args){
    let arg = Array.prototype.slice.call(arguments,1);
    const that = this;
    return function(){
       args = arg.concat(Array.prototype.slice.call(arguments));
       return that.apply(obj,args);
    }
}
const o = {
    name:'yyx',
    age:'20',
    getAge(){
        console.log(this.age);
    }
}
const getAge = o.getAge;
const bindGetAge = getAge.bindDemo(o)
bindGetAge()    //20

```

### 将bind函数绑定在原型链上

上面的函数已经可以解决一些实际问题,但依旧存在一些问题。当**thisArg**为原型链上的对象时,bind对象需要能调用原型上的方法。

```javascript
Function.prototype.bindDemo = function(obj,args){
    let arg = Array.prototype.slice.call(arguments,1);
    const that = this;
    let tmp =  function(){
       args = arg.concat(Array.prototype.slice.call(arguments));
       return that.apply(obj,args);
    }
    tmp.prototype = this.prototype;
    return tmp;
}
const o = {
    name:'yyx',
    age:'20',
    getAge(){
        console.log(this.age);
    }
}
const getAge = o.getAge;
const bindGetAge = getAge.bindDemo(o)
bindGetAge()   //20

```



