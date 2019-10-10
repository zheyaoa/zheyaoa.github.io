---
title: 前端面试遇到问题的一些总结
date: 2019-08-18 17:40:50
tags: [面试总结]
categories: [JavaScript]
---

一些遇到的面试题的总结。

<!-- more -->

### 盒模型

- 什么是盒模型：盒模型又称框模型（Box Model）,包含了元素内容（content）、内边距（padding）、边框（border）、外边距（margin）几个要素。

![](https://segmentfault.com/img/remote/1460000013069519)

- 常规盒模型 与 IE 盒模型的区别

  IE模型和标准模型唯一的区别是内容计算方式的不同，如下图所示：

![图2.IE模型宽度计算示意图](https://segmentfault.com/img/remote/1460000013069520)

  	**IE模型元素宽度width=content+padding**，高度计算相同

![图3.标准模型计算示意图](https://segmentfault.com/img/remote/1460000013069521)

​	**标准模型元素宽度width=content**，高度计算相同

- #### css如何设置获取这两种模型的宽和高

  通过css3新增的属性 `box-sizing: content-box | border-box`分别设置盒模型为标准模型（`content-box`）和IE模型（`border-box`）。

  ```
  .content-box {
    box-sizing:content-box;
    width: 100px;
    height: 50px;
    padding: 10px;
    border: 5px solid red;
    margin: 15px;
  }
  ```

  ![图4.标准模型实类图](https://segmentfault.com/img/remote/1460000013069522)

  `.content-box`设置为标准模型，它的元素宽度width=100px。

  ------

  ```
  .border-box {
    box-sizing: border-box;
    width: 100px;
    height: 50px;
    padding: 10px;
    border: 5px solid red;
    margin: 15px;
  }
  ```

  ![图5.IE模型实类图](https://segmentfault.com/img/remote/1460000013069523)

  `.border-box`设置为IE模型，它的元素宽度width=content + 2 *padding + 2* border = 70px + 2 *10px + 2* 5px = 100px。

### 闭包

- 什么是闭包

  一个持有外部环境变量的函数就是闭包

  引用轮子哥的话

  ![](https://pic2.zhimg.com/80/9a4e99dd5fa93171c2f8f09cf3f6ff54_hd.jpg)

  

- 用途
  - 封装（私有作用域）
  - 匿名自执行函数
  - 结果缓存

### position属性

元素在页面中的布局遵守一套文档流的方式，默认的定位属性值为`static`。它其实是未被设置定位的。

元素如果被定位了，那么它的`top,left,bottom,right`值就会生效，能设置定位的属性是`relative`,`absolute`和`fixed`。

需要注意的另一点是被定位的元素层次(`z-index`)会得到提高。

- relative（相对定位）

设置了相对定位之后，通过修改`top,left,bottom,right`值，元素会在自身文档流所在位置上被移动，其他的元素则不会调整位置来弥补它偏离后剩下的空隙。

- absolute（绝对定位）

设置了绝对定位之后，元素脱离文档流，其他的元素会调整位置来弥补它偏离后剩下的空隙。元素偏移是相对于是它最近的设置了定位属性（`position`值不为static）的元素。

且如果元素为块级元素（`display`属性值为`block`)，那么它的宽度也会由内容撑开。因为：

> 默认文档流中块级元素如果没有设置宽度属性，会自动填满整行。

- fixed(固定定位)

设置了固定定位之后，元素相对的偏移的参考是可视窗口，即使页面滚动，元素仍然会在固定位置。

### BFC

BFC(Block formatting context)直译为"块级格式化上下文"。它**是一个独立的渲染区域**，只有**Block-level box**参与（在下面有解释）， 它规定了内部的Block-level Box如何布局，并且与这个区域外部毫不相干。

- 触发BFC的条件
  - html  根元素
  - float 元素: float 为none 以外的元素
  - 绝对定位元素: position 为 absolute|fixed 的元素
  - display 为 inline-block table-cell flex 的元素
  - overflow 除了为visiable 以外的值（scroll hidden auto） 
- BFC 应用及其特性
  - 同一个 BFC 下外边距会发生折叠
  - BFC 可以包含浮动的元素（清除浮动）
  - BFC 可以防止元素被浮动元素覆盖

### rem 与 em 区别

- rem: 相对的只是HTML根元素 的 font-size 大小
- em: 相对继承的父元素的 font-size 大小

### JavaScript 垃圾回收

- 引用计数垃圾收集

> 这是最初级的垃圾收集算法。此算法把“对象是否不再需要”简化定义为“对象有没有其他对象引用到它”。如果没有引用指向该对象（零引用），对象将被垃圾回收机制回收。

实例：

```javascript
var o = { 
  a: {
    b:2
  }
}; 
// 两个对象被创建，一个作为另一个的属性被引用，另一个被分配给变量o
// 很显然，没有一个可以被垃圾收集


var o2 = o; // o2变量是第二个对“这个对象”的引用

o = 1;      // 现在，“这个对象”的原始引用o被o2替换了

var oa = o2.a; // 引用“这个对象”的a属性
// 现在，“这个对象”有两个引用了，一个是o2，一个是oa

o2 = "yo"; // 最初的对象现在已经是零引用了
           // 他可以被垃圾回收了
           // 然而它的属性a的对象还在被oa引用，所以还不能回收

oa = null; // a属性的那个对象现在也是零引用了
           // 它可以被垃圾回收了
```

该算法有个限制：无法处理循环引用的事例。在下面的例子中，两个对象被创建，并互相引用，形成了一个循环。它们被调用之后会离开函数作用域，所以它们已经没有用了，可以被回收了。然而，引用计数算法考虑到它们互相都有至少一次引用，所以它们不会被回收。

-  标记清除算法

### 重绘和重排

- 当页面只是样式发生了改变时,会发生重绘。
- 当页面布局发生了改变时,会进行重排。

### Vue 和 React 的区别

- Vue是双向数据流，React是单向数据流
- Vue使用mixin创建高阶组件，React使用推荐使用HOC

### 跨域解决方案

> 常见跨域场景:不同域名,同一域名不同端口,主域相同，子域不同,同一域名，不同协议

* jsonp 利用了script可以跨域的原理
* CORS
* nginx
* node中间件

### Array.prototype.flat

可指定打平层数的flat函数

```javascript
Array.prototype.flat = function(index = 1){
    const rs = [];
    this.forEach(item => {
        if(index>0&&Array.isArray(item)){
            rs.push(...item.flat(index-1));
        }else{
            rs.push(item)
        }
    })
    return rs;
}
var demo = [1,[2,3,4,[5,6]],[7,8]]
console.log(demo.flat())  //[ 1, 2, 3, 4, [ 5, 6 ], 7, 8 ]
```

### 深拷贝

```javascript
function DeepCopy(obj){
    // 循环引用
    const map = new Map();
    function dp(obj){
        if(map.has(obj)){
            return map.get(obj);
        }
        const rs ={};
        map.set(obj,rs);
        Object.entries(obj).forEach(([key,value]) => {
            if(value&&typeof value === 'object'){
                rs[key] = dp(value);
            }else{
                rs[key] = value;
            }
        })
        return rs;
    }
    return dp(obj)
}
var A = {
    B: {
        name: 'b'
    },
    C: {
        name: 'c'
    },
    D: {

    }
};
A.D.E = A.B;


var X = DeepCopy(A);
console.log(X.B); // 输出： {name: "b"}
console.log(X.D.E);// 输出: {name: "b"}
console.log(X.B === X.D.E); // 输出： true
```

