---
layout: layout
title: JavaScript创建对象
tags: Object
categories: JavaScript
date: 2019-01-22 21:08:45
---


> JavaScript 中可以说一切都是对象，这里复习了一下如何去创建一个JavaScript对象。

<!-- more -->

### JavaScript 创建对象

> 在JavaScript中所有的事物都是对象，例如String,Date,数值等等,对象只是带有属性和方法的特殊数据类型。此外JavaScript允许自定义对象。下面是创建对象的一些方法

###Object 构造函数

> 使用new 操作符跟Object构造函数,代码如下

```javascript
var person = new Object();
person.name = 'zheyao';
person.age = 20
```

###对象字面量创建对象

> 对象字面量是对象定义的一种简写方式，代码如下

```javascript
var person = {
    name:'zheyao',
    age:20
}
```

虽然上面两种方法都可以创建创建单个对象，但这些方法又一个巨大的缺点。使用一个对象创建多个有相同属性的对象，会产生大量的重复代码。下面的方法将解决这些问题。

###工厂模式

> 工厂模式是一种广为认知的设计模式，这种模式抽象了创建对象的具体过程。使用函数封装创建指定对象的细节。代码如下

```javascript
function createPerson(){
    var o = new Object();
    o.name = 'zheyao';
    o.age = 20;
    return 0;
}
```

工厂模式解决了创建多个相似代码的问题，却 没有带来对象识别的问题。下面描述的函数构造模式很好的解决了这个问题 。

###构造函数模式

> 构造函数模式可用于创建特定类型的对象。创建自定义的构造函数，从而可以创建自定义对象的类型，属以及方法。代码如下

```javascript
function Person(name,age){
    this.name = name;
    this.age = age;
    this.sayName = function(){
        console.log(this.name)
    }
}
var person1 = new Person('zheyao',20);
var person2 = new Person('yw',19);
```

在函数构造模式模式中，并没有显示的创建对象。当执行代码**var o = new Person('zheyao',20)**时，我们可以理解为进行了如下操作

```javascript
var o = new Object();
o = Person().call(o,'zheyao',20)
```

函数构造模式解决了对象识别的问题 ，但是也同样出现了一个新的问题 ，当我们创建两个新的对象person1,person2时。构造函数中的sayName函数被实例化了两次。原型模式解决了这个问题。

###原型模式

>我们创建的么一个函数都有一个**原型(prototype)**属性,可以用这个属性来共享一些相同类型的公共函数和属性，代码如下

```javascript
function Person(){}
Person.prototype.name = 'zheyao';
Person.prototype.age = 20;
Person.prototype.sayName = function(){
    console.log(this.name)
}
```

原型模式的好处是可以共享属性，换句话说不必在构造函数中定义对象实例的信息，而是可以将这些将这些信息直接添加到原型对象。但原型模式也不是没有缺点，对于原型链的引用类型来说，问题就比较突出，代码如下。

```javascript
function Person(){};
Person.prototype = {
    constructor:Person,
    colors:['red','black']
}
var p1 = new Person();
var p2 = new Person();
p1.colors.push('orange');

console.log(p1.colors) //'red','black','orange'
console.log(p2.colors) //'red','black','orange'
```

可以看出当我们给对象p1的colors属性添加了一个值‘orange'时，对象p2的colors属性随之改变。这是因为colors并不是对象p1的实例属性。而是储存在p1的原型链中的。而对象p1,p2公用一个原型链。所以当属性在p1进行改变时。在p2也会随之改变。下面这种模式解决了这个问题

* 组合使用构造函数模式与原型模式

> 创建对象的最常见方式,就是组合使用构造函数模式与原型模式。使用构造函数创建属于自己的实例属性。原型模式创建一些公共的属性。代码如下

```javascript
function Person(name,age){
	this.name = name;
    this.age = age;
}
Person.prototype.sayName  = function(){
    console.log(this.name)
}
```

这种方法可以规避两种模式的缺点，是JavaScript中最常用的方法。