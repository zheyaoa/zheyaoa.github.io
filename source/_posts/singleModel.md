---
title: 设计模式--单例模式
date: 2019-01-31 17:00:06
tags: 设计模式
categories: [JavaScript,设计模式]
---

> 单例模式是一种常用的模式,有一些对象我们往往只需要一个。例如全局缓存，浏览器的window对象，在JavaScript中，单例模式的作用依旧广泛。例如实现一个唯一的悬浮窗，模拟JQuery/Vue的one函数。

<!-- more -->

### 实现单例模式

> 单例模式的实现并不复杂，其核心的原理便是利用一个变量查看该对象是否已被构造,若被构造则返回该实例。否则创建一个 新的实例并返回。

```javascript
var Singleton = function(name){
    this.name = name;
}
Singleton.prototype.getInfo = function(){
    console.log(this.name)
}
Singleton.getInstance = (function(){
    instance = null;
    return function(name){
    	return instance||instance = new Singleton(name)
    }
})()
var a = Singleton.getInstance('a');
var b = Singleton.getInstance('b');
console.log(a===b)    //true
```

上面实现了一个简单的单例模式,我们通过Singleton.getInstance获取Singleton对象的唯一对象，这种方法的实现很简单。但是也存在一些问题 ，使用者必须知道这是一个单例类(使用Singleton.getInstance而不是new XXX),这就增加了代码的不透明性。下面将编写透明的单例类

### 透明的单例模式

```javascript
var createSingle = (function(fn){
    var result;
    return function(){
        return result||result=fn.apply(this,arguments);
    }
})()
function Person(name){
    this.name = name;
}
var createSinglePerson = createSingle(function (name){
    return new Person(name)
});

var a = new createSinglePerson('a');
var b = new createSinglePerson('b');

console.log(a==b)   //true
```

在这里我们构造了一个透明的单例模式(通过new XXX 获取对象)，同时使用了代理模式,当要创建其他的单例类时，只需要稍微进行修改。

```javascript
//创建单例类Student
function Student(name,age){
    this.name = name;
    this.age = age;
}
var createSingleStudent = createSingle(function(name,age){
    return new Student(name,age)
})
```

这样我们就获取了一个获取Student的单例类createSingleStudent.

### 单例模式的实际应用

* 使用单例模式实现JQuery中的one 功能

```javascript
var bindEvent = getSingle(function(){
    document.getElementById('div1').onclick = function(){
        console.log('click')
    }
    return true;
})
```