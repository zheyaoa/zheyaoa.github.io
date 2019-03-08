---
title: 设计模式-发布/订阅者模式
date: 2019-02-20 23:47:13
tags: 设计模式
categories: [JavaScript,设计模式]
---

> 发布/订阅者模式,是我们在生产环境中常用的一种模式，他的运用非常的丰富。在我们常用框架Vue,短信的订阅与发布等等。都有着发布/订阅者模式的身影。

<!-- more -->

### 什么是发布/订阅者模式 

​	发布/订阅者模式，这个模式的作用是一个把订阅者与发布者连接在一起的代理。发布者是当完成某些过程时触发时间的对象，订阅者是希望发布者发布时被通知的对象。

​	举一个生活中常用的例子，比如现在有一个人（发布者）能够正确分析出股票的情况，每天都把股票趋势发布给需要该信息的人，但是他不知道有哪些人（订阅者）想获取该信息，甚至说有某些人不再需要该信息了（取消订阅）。当面对这样的事件时，我们会想到的方法就是依靠中介。发布/订阅者模式就是一个这样的中介，发布者每天定时把股票的趋势发送给中介,想获取该信息的人只需要去该中介订阅该信息，当发布者要求中介发布消息时（完成了某些事件），所以的订阅者都可获得信息。

发布/订阅者模式的好处在于

- 松耦合:发布者和订阅者松耦合，甚至可以不需要知道对方的存在。发布者与订阅者依靠代理完成信息的传递。

### 发布/订阅者模式的简单JavaScript实现

```javascript
//Observer.js 
function Observer(){
    this.callbackList = [];
}
//添加订阅者
Observer.prototype.listen = function(fn){
    this.callbackList.push(fn);
}
//发布消息
Observer.prototype.triger = function(){
    this.callbackList.forEach(fn=>{
        fn.apply(this,arguments)
    })
}


//demo
var observer = new Observer();
//用户user1订阅信息
observer.listen(function(){
    console.log('给user1发送短信')
});
//用户user2订阅信息
observer.listen(function(){
    console.log('给user2发送短信')
});
//中介发布消息
observer.triger();
##给user1发送短信
##给user2发送短信
```

上面我们就简单实现了一个简单的发布/订阅者模式，我们可以发现listen监听的是一个函数。这与我们们的预期订阅一个对象不太符合。这是因为JavaScript的特性函数可以作为参数的原因。下面将上面的发布/订阅者模式改写为对象版本。



### 发布/订阅者模式的面向对象版本实现

```javascript
function Sub(){
	this.subs = [];
}
Sub.prototype.addSub = function(sub){
    this.subs.push(sub);
}
Sub.prototype.notify = function(){
    this.subs.forEach(sub =>{
        sub.update()
    })
}

//订阅者原型
function Watcher(fn){
    this.fn = fn;
}
Wactcher.prototype.update = fuction(){
    this.fn();
}



//demo
//基于订阅者原型创建新对象
function User(name){
    this.name = name;
}
User.prototype = new Watch();
User.prototype.fn = function(){
	console.log(`向 ${this.name} 发送短信`)
}
User.prototype.construct = User;

var sub = new Sub();
sub.addSub(new User('yyx'))
sub.addSub(new User('zheyao'))
sub.notify()
//向yyx发送短信
//向zheyao发送短信
```

可以看出在面向对象的发布订阅者模式中,我们构造了一个原型Watcher，然后基于Watcher创建了新对象User，在Sub中储存订阅者User的信息。实现了文章开头所需要的功能。 同时我们也能基于该模型添加更多如撤销订阅者的操作。