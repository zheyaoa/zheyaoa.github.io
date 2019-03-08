---
title: 手写一个Promise/A规范的Promise
date: 2019-03-05 15:58:25
tags: [Promise]
categories: [JavaScript,ES6]
---

> 最近比较闲，忙里偷闲看了下Promise/A+标准，参考了一些文章手动实现了一个Promise。为了加深印象，在这里做一个总结。也供大家参考

<!-- more -->

### 什么是Promise

​	Promise是一种异步解决方案，比起传统的回调更加的方便与强大。避免了多重异步函数嵌套产生所谓的回调地狱。ES6将其写进了语言标准，统一了语法。这篇文章不会过多的讲解Promise的语法和特性（若不了解可以先查看阮一峰老师的[Promise教程](http://es6.ruanyifeng.com/#docs/promise)），而会专注于Promise底层的实现。实现一个符合Promise/A+规范的promise demo.

​	在这里简单创建一个Promise实例

```javascript
let promise = new Promise((reslove,reject)=>{
    setTimeout(()=>{
        reslove('success')
    },100)
    console.log('this is a promise demo')
})
promise.then((data)=>{
	console.log(data)
})


//output
this is a promise demo
success
```

上面是一个Promise 实例，输出内容this is a promise demo 然后一秒后输出success 可以看出，promise 很好的解决了异步的问题。

### Promise/A+ 规范

[Promise/A+](https://link.juejin.im/?target=https%3A%2F%2Fpromisesaplus.com%2F)规范扩展了早期的[Promise/A proposal](https://link.juejin.im/?target=http%3A%2F%2Fwiki.commonjs.org%2Fwiki%2FPromises%2FA)提案，我们来解读一下Promise/A+规范。

#### 术语

***

* **promise:**promise 是一个拥有 `then` 方法的对象或函数，其行为符合本规范；

* **thenable**:是一个定义了 `then` 方法的对象或函数，文中译作“拥有 `then` 方法”；
* **value**:指任何 JavaScript 的合法值（包括 `undefined` , thenable 和 promise）

* **exception**:是使用throw抛出的一个异常

* **reason**:表示一个 promise 的拒绝原因。

#### 要求

***

* **promise的状态**:一个 Promise 的当前状态必须为以下三种状态中的一种：**等待态（Pending）**、**执行态（Fulfilled）**和**拒绝态（Rejected）**。

* **等待态**（pending）:

  处于等待态时，promise 需满足以下条件：

  * 可以迁移至执行态或拒绝态

* **执行态** （Fulfilling）

  处于执行态时，promise 需满足以下条件：

  * 不能迁移至其他任何状态
  * 必须拥有一个**不可变**的终值

* **拒绝态**（Rejected）

  处于拒绝态时，promise 需满足以下条件：

  * 不能迁移至其他任何状态

  * 必须拥有一个**不可变**的据因

* **Then方法**

  一个 promise 必须提供一个 `then` 方法以访问其当前值、终值和据因。

  promise 的 `then` 方法接受两个参数：

```javascript
promise.then(onFulfilled, onRejected)
```

​	then方法的返回值必须是一个Promise

#### 其他

***

其他规范就不再过多的阐述了，若想详细了解可以查看图灵社区的[Promise/A+规范](http://www.ituring.com.cn/article/66566)

### Promise的初步实现

> 从Promose/A+ 规范和日常使用中我们可以知道下面几点。

* Promise 是一个类，当执行``new Promise()``时会返回一个Promise实例
* 每个Promise实例上都有一个then方法，参数分别为执行成功/执行失败时的回调。当Promise实例从等待态改变为其他状态时，执行相应的回调函数。
* 一个 Promise 的当前状态必须为以下三种状态中的一种：**等待态（Pending）**、**执行态（Fulfilled）**和**拒绝态（Rejected）**。

* Promise的默认状态是等待态，可以改变为执行态和拒绝态。当状态改变后不会再改变为其他状态

写到这里我们基本可以实现一个Promise的原型了

```javascript
function Promise(handle){
    if(typeof handle !== 'function'){
        throw new Error("arguments need be a function")
    }
    this.state = 'pending'//state 表实例状态
    this.value = '';	//成功的值
    this.reson = '';	//失败的值
    const resolve = (value)=>{
        if(this.state == 'pending'){
        	this.state = 'resolve'
        	this.value = value;
        }
    }
    const reject = (reason)=>{
        if(this.state == 'pending'){
            this.state = 'reject'
            this.reason = reason
        }
    }
    try{//处理handle错误
        handle(resolve,reject)
    }catch(e){
        reject(e)
    }
}
/**
*@param {*} onResolve 状态为reslove的回调函数
*@param {*} onReject 状态为reject 的回调函数
*/
Promise.prototype.then = function(onResolve,onReject){
    if(this.state === 'reslove'){
        onResolve(this.value)
    }else if(this.state === 'reject'){
        onReject(this.reason)
    }
}
```

测试一下

```javascript
let promise = new Promise((resolve,reject) => {
        resolve(20)
    	console.log('test one')
})
promise.then(data => {
    console.log(data)
})

//output
test one 
20
```

可以看出原型实现了我们的想要的预期效果，但是也同样也存在一些问题：只有执行同步任务时promise才能正常工作。当执行异步代码时将失效。在这里举一个例子。

```javascript
let promise = new Promise((resolve,reject) => {
	setTimeout(()=>{
    	resolve(20)
    	console.log('test one')
	},100)
})
promise.then(data => {
    console.log(data)
})

//output
test one 

```

我们知道Promise解决的主要是异步问题，下面将实现一个可以执行异步调用的Promise

### Promise的异步实现

> 异步调用时，当我们执行promise的then函数时，promise可能仍处于pending状态。这时对应的回调函数不能被执行。为了解决这个问题我们给Promise对象添加了两个回调函数数组，当promise实例处于pending状态时将回调函数加入回调函数数组。当状态发生改变时再进行执行

话不多说，上代码

```javascript
function Promise(handle){
    if(typeof handle !== 'function'){
        throw new Error("arguments need be a function")
    }
    this.state = 'pending'//state 表实例状态
    this.value = '';	//成功的值
    this.reson = '';	//失败的值
    this.onResolveCallbacks = [];
    this.onRejectCallbacks = [];
    const resolve = (value)=>{
        if(this.state == 'pending'){
        	this.state = 'resolve'
        	this.value = value;
            this.onResolveCallbacks.forEach(fn => fn())
        }
    }
    const reject = (reason)=>{
        if(this.state == 'pending'){
            this.state = 'reject'
            this.reason = reason
            this.onRejectCallbacks.forEach(fn => fn())
        }
    }
    try{//处理handle错误
        handle(resolve,reject)
    }catch(e){
        reject(e)
    }
}
/**
*@param {*} onReslove 状态为reslove的回调函数
*@param {*} onReject 状态为reject 的回调函数
*/
Promise.prototype.then = function(onResolve,onReject){
    if(this.state === 'reslove'){
        onReslove(this.value)
    }else if(this.state === 'reject'){
        onReject(this.reason)
    }else if(this.state === 'pending'){
        this.onResolveCallbacks.push(() => {
            onReslove(this.value)
        })
        this.onRejectCallbacks.push(() => {
            onReject(this.reason)
        })
    }
}
```

做个测试

```javascript

const promise = new PromiseAsync((resolve,reject) => {
    setTimeout(() => {
        reslove(20)
    },100)
    console.log('test 2')
})
promise.
then(data => {
    console.log(data)
}) 

//output
test2
20
```

### 链式调用的实现

​	这样下来，我们就实现了异步的Promsie。但是根据Promise规范可知，promise的then函数会返回一个promise对象，这样才能够实现promise的链式调用。用过jQuery的人都知道，jQuery依靠``return this`` 实现了链式调用，那么promise也是依靠返回this实现链式调用吗。

​	答案显然是否认的，promise then 函数的每一次调用，都会返回一个全新的promise实例。下面将实现promise的链式调用

```javascript
Promise.prototype.then = function(onResolve,onReject){
    const promise = new Promise((reslove,reject) =>{
        if(this.state === 'reslove'){
            onResolve(this.value)
        }else if(this.state === 'reject'){
            onReject(this.reason)
        }else if(this.state === 'pending'){
            this.onResolveCallbacks.push(() => {
                onReslove(this.value)
            })
            this.onRejectCallbacks.push(() => {
                onReject(this.reason)
            })
    	}
    })
    return promsie
}
```

#### 链式调用的错误处理

在Promise/A+ 规范中，对错误处理有这样一段描述

对于 `` promise2 = promise1.then(onReslove,onReject)  ``中，如果 `onFulfilled` 或者 `onRejected` 抛出一个异常 ``e`` ，则 ``promise2`` 必须拒绝执行，并返回拒因 ``e``。我们在代码中加上对错误的捕获。

```javascript

PromiseComplete.prototype.then = function(onReslove,onReject){ 
    let promise;
        promise = new PromiseComplete((reslove,reject)=>{
        let x;
        if(this.state === 'reslove'){
            handleError(() => {
            x = onReslove(this.value);
            },reject)
        }else if(this.state === 'reject'){
            handleError(() => {
                x = onReject(this.reason);
            },reject)
        }else if(this.state === 'pending'){
            this.onResloveCallbacks.push(()=>{
                handleError(() => {
                    x = onReslove(this.value)
                },reject)
            })
            this.onRejectCallbacks.push(() => {
                handleError(() => {
                    onReject(this.reason)
                },reject)            
            })
        }
    })
    return promise
}

function handlerErr(fn,reject){
    try{
        fn()
    }catch(e){
      	reject(e)
    }
}
```

​	这样就完成了对 Promise 链式操作的事件捕获

#### Promise的传递

​	最后我们需要实现Promise实现中，最难实现的部分。关于这部分的规范可以在[Promise/A+规范](http://www.ituring.com.cn/article/66566)

Promise 解决过程中看到。大致部分如下

​	    如果 `onFulfilled` 或者 `onRejected` 返回一个值 `x` ，则运行下面的 **Promise 解决过程**：`[[Resolve]](promise2, x)`

* 如果 `promise` 和 `x` 指向同一对象，以 `TypeError` 为据因拒绝执行 `promise`

* 如果 `x` 为 Promise ，则使 `promise` 接受 `x` 的状态 
* 如果 `x` 为对象或者函数：则将x视为promise对待，若在执行过程中出现错误e，则reject该错误。

```javascript
handleError(() => {
    //x是上一个promise的返回值
    x = onReslove(this.value);
    //将返回值x传入promiseReslove中
    	promiseReslove(promise,x,reslove,reject);
},reject)
```

下面实现promiserReslove 处理promise的解决问题。

```javascript
/**
 * 
 * @param {*} promise 下一次then返回的promise实例 
 * @param {*} x       本次返回值x
 * @param {*} reslove promise2的成功方法
 * @param {*} reject  promise2的失败方法
 */
const promiseReslove = (promise,x,reslove,reject) => {
    //x 与 promise 相等
    if(promise === x ){
        return reject(new TypeError("循环引用"))
    }
    //x 是一个 Promise 实例
    if(x instanceof Promise){
        //x的属性未知，用promiseReslove继续判断
        x.then((value => {
            promiseReslove(promise,value,reslove,reject)
        }),reject)
        return
    }
    //x是一个thenable对象
    if(x!== null&&(typeof x === 'object'||typeof x === 'function')){
        let then = x.then,isCalled = false;
        try{
            //获取then函数,根据Promise规范，resolve或reject只允许执行一次
            if(typeof then === 'function'){
                then.call(x,y=>{
                    if(isCalled) return;
                    isCalled = true;
                    promiseReslove(promise,y,reslove,reject)
                },r => {
                    if(isCalled) return
                    isCalled = true;
                    reject(r)
                })
            }else{
                reslove(x)
            }
        }catch(e){
            if(isCalled) return;
            isCalled = true;
            reslove(e)
        }
    //若x 是一个普通值，则直接用resolve返回
    }else{
        reslove(x)
    }
}
```

老规矩 测试一下

```javascript
const promise = new Promise((reslove,reject)=>{
    reslove(20)
})
promise.then(data => {
    return new Promise((reslove,reject) => {
        setTimeout(()=>{
            reject(data)
        },100)
    })
})
.then(undefined,err =>{
    console.log(err)
})

//outout 
20
```

#### 值的穿透

​	以上，我们的promise好像已经差不多了，但是还有一个问题，需要处理。源码可以在hen中实现什么都不传。promise中管这种现象叫，``值的穿透``。 因此，我们需要在then方法里，对then方法的入参进行容错处理：

```javascript
Promise.prototype.then = function(onReslove,onReject){
    onReslove = typeof onReslove === 'function'?onReslove:() => this.value 
    onReject = typeof onReject === 'function'?onReject:()=>{throw new Error(this.reason)}
    ...
}
```

测试一次

```javascript
const promise = new Promise((reslove,reject)=>{
    reslove(20)
})
promise.
then()
.then(data => {
    console.log(data)
})

//output
20
```

另外，promise规范中要求，所有的`onFufilled`和`onRejected`都需要异步执行，如果不加异步可能造成测试的不稳定性，所以我们给执行这两个方法执行的地方都加上异步方法。个人理解异步执行其实已经被callbacks回调解决。但是为了确保测试的稳定性，在这里给``onFufilled``和``onRejected``加上定时器

```javascript
const reslove = (value) => {
    setTimeout(()=>{
        if(this.state === 'pending'){
            this.state = 'reslove';
            this.value = value;
            this.onResloveCallbacks.forEach(fn => {
                fn();
            })
        }
    },0)
}
const reject = (reason) => {
    setTimeout(()=>{
        if(this.state == 'pending'){
            this.state = 'reject';
            this.reason = reason;
            this.onRejectCallbacks.forEach(fn => {
                fn();
            })
        }
    },0)
}
```

以上，promise/A+ 规范 的实现就全部完成。

### 完整代码

```javascript
//Promise A+ 
function Promise(handle){
    if(typeof handle != 'function'){
        throw new Error("handle need be a function")
    }
    this.state = 'pending';
    this.value = undefined;
    this.resson = undefined;
    this.onResloveCallbacks = [];
    this.onRejectCallbacks = [];
    const reslove = (value) => {
        setTimeout(()=>{
            if(this.state === 'pending'){
                this.state = 'reslove';
                this.value = value;
                this.onResloveCallbacks.forEach(fn => {
                    fn();
                })
            }
        },0)
    }
    const reject = (reason) => {
        setTimeout(()=>{
            if(this.state == 'pending'){
                this.state = 'reject';
                this.reason = reason;
                this.onRejectCallbacks.forEach(fn => {
                    fn();
                })
            }
        },0)
    }
    try{
        handle(reslove,reject)
    }catch(e){
        reject(e)
    }
}
Promise.prototype.then = function(onReslove,onReject){
    onReslove = typeof onReslove === 'function'?onReslove:() => this.value 
    onReject = typeof onReject === 'function'?onReject:()=>{throw new Error(this.reason)} 
    let promise;
        promise = new PromiseComplete((reslove,reject)=>{
        let x;
        if(this.state === 'reslove'){
            handleError(() => {
            x = onReslove(this.value);
                promiseReslove(promise,x,reslove,reject);
            },reject)
        }else if(this.state === 'reject'){
            handleError(() => {
                x = onReject(this.reason);
                promiseReslove(promise,x,reslove,reject);
            },reject)
        }else if(this.state === 'pending'){
            this.onResloveCallbacks.push(()=>{
                handleError(() => {
                    x = onReslove(this.value)
                    promiseReslove(promise,x,reslove,reject)
                },reject)
            })
            this.onRejectCallbacks.push(() => {
                handleError(() => {
                    onReject(this.reason)
                    promiseReslove(promise,x,reslove,reject)
                },reject)            
            })
        }
    })
    return promise
}
const handleError = (fn,reject)=>{
    try{
        fn()
    }catch(e){
        reject(e)
    }
}
/**
 * 
 * @param {*} promise 下一次then返回的promise实例 
 * @param {*} x       本次返回值x
 * @param {*} reslove promise2的成功方法
 * @param {*} reject  promise2的失败方法
 */
const promiseReslove = (promise,x,reslove,reject) => {
    if(promise === x ){
        return reject(new TypeError("循环引用"))
    }
    if(x instanceof PromiseComplete){
        x.then((value => {
            promiseReslove(promise,value,reslove,reject)
        }),reject)
        return
    }
    if(x!== null&&(typeof x === 'object'||typeof x === 'function')){
        let then = x.then,isCalled = false;
        try{
            if(typeof then === 'function'){
                then.call(x,y=>{
                    if(isCalled) return;
                    isCalled = true;
                    promiseReslove(promise,y,reslove,reject)
                },r => {
                    if(isCalled) return
                    isCalled = true;
                    reject(r)
                })
            }else{
                reslove(x)
            }
        }catch(e){
            if(isCalled) return;
            isCalled = true;
            reject(e)
        }
    }else{
        reslove(x)
    }
}
module.exports = Promise;
```

