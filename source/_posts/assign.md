---
title: ES6下JavaScript对象方法Object.assign()
date: 2019-05-16 23:03:53
tags: Object
categories: [JavaScript,ECMAScript6]
---

，所以最近看了看[MDN_JavaScript](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create)板块下的一些知识。这里总结一下`Object.assign()`的用法

，所以最近看了看MDN_JavaScript板块下的一些知识。这里总结一下Object.assign()的用法

最近看一些框架，组件的实现，发现有些地方不是很能理解(原生js学的太差了)，所以最近看了看MDN_JavaScript板块下的一些知识。这里总结一下Object.assign()的用法

<!-- more --> 

-----

### Object.assign(target,...sources)

> target: 目标对象
>
> sources:  源对象

* **Object.assign()**方法用于将所有**可枚举属性**的值从一个对象或多个原对象复制到目标对象。同时将返回目标对象。

  ```javascript
  const obj = { a: 1 }
  const copy = Object.assign({},obj)
  console.log(copy) //{a:1}
  ```

* **Object.assign()**方法可以复制Symbol属性

  ```javascript
  const o1 = {a:1};
  const o2 = { [Symbol('foo')]:2}
  const o3 = Object.assign(o1,o2)
  console.log(o3)  //{a:1, [Symbol('foo')]:2} 
  ```

* **Object.assign()** 进行的是**浅拷贝**,如果说源对象的某个属性值是对象，那么目标对象拷贝得到的是这个对象的引用

  ```javascript
  let obj1 = { a: 0 , b: { c: 0}}; 
  let obj2 = Object.assign({}, obj1); 
  
  obj1.b.c = 1
  console.log(obj2.b.c)  //2
  ```

* **Object.assign()**不能拷贝**继承属性**和**不可枚举属性**

  ```javascript
  const obj = Object.create({foo: 1}, { // foo 是个继承属性。
      bar: {
          value: 2  // bar 是个不可枚举属性。
      },
      baz: {
          value: 3,
          enumerable: true  // baz 是个自身可枚举属性。
      }
  });
  
  const copy = Object.assign({}, obj);
  console.log(copy); // { baz: 3 }
  ```

*  **Object.assign()**合并相同的属性，会被后续参数中具有相同属性的其他对象覆盖 

  ```javascript
  const o1 = { a: 1, b: 1, c: 1 };
  const o2 = { b: 2, c: 2 };
  const o3 = { c: 3 };
  
  const obj = Object.assign({}, o1, o2, o3);
  console.log(obj); // { a: 1, b: 2, c: 3 }
  ```

* 原始类型会被包装成对象

   ```javascript
     const v1 = "abc";
     const v2 = true;
     const v3 = 10;
     const v4 = Symbol("foo")
     const obj = Object.assign({}, v1, null, v2, undefined, v3, v4); 
     // 原始类型会被包装，null 和 undefined 会被忽略。
     // 注意，只有字符串的包装对象才可能有自身可枚举属性。
     console.log(obj); // { "0": "a", "1": "b", "2": "c" }
   ```

### Polyfill

> 此polyfill不支持symbol,因为ES5中没有Symbol

   ```javascript
if(typeof Object.assign != 'function'){
    Object.defineProperty(Object,'assign',{
        value:function(target,sources){
            'use strict'
            if(target == null){
                throw new TypeError('Cannot convert undefined or null to Object')
            }
            let to = Object(target);
            for(var index=1;index<arguments.length;index++){
                var nextSource = arguments[index];
                if(nextSource != null){
                    for(let nextkey in nextSource){
                        if(Object.prototype.hasOwnProperty.call(nextSource,nextkey)){
                            to[nextkey]=nextSource[nextkey];
                        }
                    }
                }
            }
            return to;
        },
        configurable:true,
        writable:true
    })
}

   ```













