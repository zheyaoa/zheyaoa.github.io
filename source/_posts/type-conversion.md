---
title: JavaScript类型转换
date: 2019-09-30 11:25:26
tags: [面试总结]
categories: [JavaScript]
---

JavaScript的类型转换在面试中是常见的坑点与考点。在这里做一个类型转化的总结:在JavaScript 中类型转化有三种情况,分别是:

- 转化为布尔值
- 转换为数字
- 转换为字符串

<!-- more -->

### 转Boolean

在条件判断时，除了`undefined`,`null`,`false`,`NaN`,`0`,`''`,`0`,`-0`，其他值都转为`true`，包括所有对象

```javascript
function Test(val){
    if(val){
        console.log(true);
    }else{
        console.log(false);
    }
}
Test('')  //false
Test(NaN)	//false
Test([])	//true
Test({})	//true
```

### 对象转原始对象

> 参考链接 : http://javascript.info/object-toprimitive

对象在转换类型的时候，会调用内置的`[[ToPrimitive]]`函数，对于该函数，算法逻辑一般如下

- 如果已经是原始类型了，那么不需要转换。
- 如果要转换成`String`类型调用`toString()`,如果转换为基础类型的话就返回转换的值,否则调用`valueOf()`
- 如果要转换为`非String`类型调用`valueOf()`,如果转换为基础类型的话直接返回，否则调用`toString()`

- 如果都没有返回原始类型，则报错

也可以重写 `Symbol.toPrimitive` ，该方法在转原始类型时调用优先级最高。

```javascript
let a = {
  valueOf() {
    return 0
  },
  toString() {
    return '1'
  },
  [Symbol.toPrimitive]() {
    return 2
  }
}
1 + a // => 3
```

### 四则运算符

加法不同于其他其他运算符,他有一下几个特点

- 如果运算中其中一方会字符串，那么就会把另一方转化为字符串。
- 如果运算中其中一方不是数字或者字符串,那么就会把它也转换成字符或字符串。

```javascript
1 + '1' // '11'
true + true // 2
4 + [1,2,3] // "41,2,3"
```

那么对于除了加法的运算符来说，只要其中一方是数字，那么另一方就会被转为数字

```javascript
4 * '3' // 12
4 * [] // 0
4 * [1, 2] // NaN
```

### 比较运算符

- 如果是对象，就通过 `toPrimitive` 转换对象

- 如果是字符串，就通过 `unicode` 字符索引来比较

```javascript
let a = {
  valueOf() {
    return 0
  },
  toString() {
    return '1'
  }
}
a > -1 // true
```

### == vs ===

对于 `==` 来说，如果对比双方的类型**不一样**的话，就会进行**类型转换**，这也就用到了我们上面讲的内容。

假如我们需要对比 `x` 和 `y` 是否相同，就会进行如下判断流程：

1. 首先会判断两者类型是否**相同**。相同的话就是比大小了

2. 类型不相同的话，那么就会进行类型转换

3. 会先判断是否在对比 `null` 和 `undefined`，是的话就会返回 `true`

4. 判断两者类型是否为 `string` 和 `number`，是的话就会将字符串转换为 `number`

   ```
   1 == '1'
         ↓
   1 ==  1
   
   ```

5. 判断其中一方是否为 `boolean`，是的话就会把 `boolean` 转为 `number` 再进行判断

   ```
   '1' == true
           ↓
   '1' ==  1
           ↓
    1  ==  1
   
   ```

6. 判断其中一方是否为 `object` 且另一方为 `string`、`number` 或者 `symbol`，是的话就会把 `object` 转为原始类型再进行判断

