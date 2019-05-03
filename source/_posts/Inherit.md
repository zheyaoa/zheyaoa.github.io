---
title: JavaScript 类的继承
date: 2019-05-01 11:56:29
tags: Object
categories: JavaScript
---

> JavaScript 作为一个面向对象的弱类型语言，同样是拥有属于自己的继承方式的。这里复习一下有关于Js的继承

<!-- more -->

###  父类的构造

首先我们构造一个父类Animals(es5)，所有的子类都将继承于父类

```javascript
function Animals(name){
    this.name = name|| 'Animals'
    //实例
    this.sleep = function(){
        console.log(this.name + "正在睡觉")
    }
}
//原型属性
Animals.prototype.eat = function(food){
    console.log(`${this.name} eat ${food}`)
}
```

### 原型链继承

ECMAScript 中描述了原型链的概念，并将原型链作为实现继承的主要方法。其基本思想是利用原型链让一个引用类型继承另一个引用类型的属性和方法。

基本模式如下

```javascript
//第一种继承方式 原型链继承
function Cat(){}
Cat.prototype = new Animals();
Cat.prototype.name = 'cat'

//test
var cat = new Cat()
console.log(cat.name)     //cat
cat.sleep()				  //cat 正在睡觉
cat.eat("fish")			  //cat eat fish
console.log(Cat.prototype.constructor)  //[Function: Animals]
console.log(cat instanceof Cat)			//true
console.log(cat instanceof Animals)		//true
```

* 在这里我们可以看到new 了一个空对象，这个空对象的prototype 指向 Animals
* 特点：基于原型链继承，既是父类的实例，也是子类的实例
* 缺点：无法实现多继承，同时父类的实例属性会被所有子类共享，成为子类的原型属性。

### 构造函数继承

为了解决原型链继承中存在的问题（多继承,父类的实例属性将成为子类的原型属性），可以使用构造函数继承。

等于是复制父类的实例属性到子类中（没有使用到prototype,即`instanceof`无效）

基本模式如下

```javascript
//构造函数继承
function Dog(name){
    Animals.call(this,name)
}

//test
const dog = new Dog('dog');

console.log(dog.name)	//dog
dog.sleep()				//!error
dog.eat("fish")			//!error
console.log(Dog.prototype.constructor) //[Function: Dog]
console.log(dog instanceof Dog)		   //true
console.log(dog instanceof Animals)	   //false
```

* 特点：可以实现多继承，可以拥有自己的实例属性。
* 缺点：可以看出 只能继承 父类Animals 的实例方法，Animals 下的原型方法 eat,sleep 都是无法继承的。即构造函数继承无法继承父类的原型方法。

### 组合继承

组合继承相当于原型链与构造函数继承的结合体，保留了他们的优点（拥有自己的的实例属性同时也能保留父类的原型属性）

基本模式如下

```javascript
function Tiger(name,age){
    Animals.call(this,name)
    this.age = age;
}
Tiger.prototype = new Animals();
Tiger.prototype.constructor = Tiger;
const tiger = new Tiger("tiger","20")
console.log(tiger.name)		//tiger
tiger.sleep()				//tiger 正在睡觉 
tiger.eat("meal")			//tiger eat meal
console.log(Tiger.prototype.constructor)  //[Function: Tiger]
console.log(tiger instanceof Tiger)		  //true
console.log(tiger instanceof Animals)	  //ture
```

* 优点:构造函数的优点是可以 同时继承父类的 实例属性和原型属性，同时使用`prototype.constructor`修正了子类的构造函数，是最常用的继承模式
* 缺点:需要调用两次父类的构造函数，第一次在原型上调用构造函数将在原型上产生实例属性name,第二次在实例中调用构造函数在实例中产生属性name,第二次产生的属性将屏蔽第一次原型的属性。

如图

![组合继承](http://cdn.zheyao.top/Animals_Tiger.png)

可以看出 在  __proto__ 属性下还有一个 name 对象，继承自 Animals 对象的 `prototype.constructor`

### 寄生组合继承

组合继承是JavaScript中最常用的继承模式，但是他也有一些缺点。组合继承最大的不足就是无论在什么情况下，都会调用两次父类的构造函数:一次在子类的原型中，另一次在子类的实例属性中。

所谓组合式继承，即通过借用构造函数来继承属性，通过原型链的的混成方式来继承方法。其思路是:不必为了子类型的原型调用父类型的构造函数，我们要的其实只是父类型原型的一个副本。同时重写子类型的`prototype.constructor` 

代码如下

```javascript
function Bird(){
    Animals.call(this,name)
    this.size = size
}
Bird.prototype = Object.create(Animals.prototype)
Bird.prototype.constructor = Bird
const bird = new Bird("bird","small")
console.log(bird.name)		//bird
bird.eat("bird food")		//bird eat bird food
console.log(bird instanceof Bird)	//true
console.log(bird instanceof Animals)	//true
console.log(bird)					//true
```

* 特点：寄生组合继承可以说是最完美的继承方式了,结合了上面几种方式的优点的同时避免了他们的缺点。
* 缺点:  逻辑比较复杂

![寄生组合继承](http://cdn.zheyao.top/Animals_Bird.png)

可以看出在 寄生组合继承模式中 __proto__ 属性下是没有  name 属性的，这是因为我们直接使 `Bird.prototype = Object.Create(Animals.prototype)  `,并没有执行父类的构造函数

### 小结

以上就是对JavaScript继承的一些总结，这里是[代码仓库](https://github.com/zheyaoa/Js/tree/master/prototype)，欢迎大家评论。



