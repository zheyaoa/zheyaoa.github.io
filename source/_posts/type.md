---
title: TypeScript的一些类型总结
date: 2019-07-14 21:25:51
tags:
---

最近一个人接手了一个公司的前端项目，是基于`Vue` + `TypeScript` 写的。其实`TypeScrip`t平时也是用过不少，一些简单的使用还是可以满足的。但是接手的项目代码高级类型漫天飞，`extends`,`implement`到处都是。没办法只能仔细看了一遍`TypeScript`文档，这里对类型做一些总结。

<!-- more -->

## 接口类型

> TypeScript的核心原则之一是对值所具有的*结构*进行类型检查。 它有时被称做“鸭式辨型法”或“结构性子类型化”。 在TypeScript里，接口的作用就是为这些类型命名和为你的代码或第三方代码定义契约。

- Interface 定义函数类型

> 接口能够描述JavaScript中对象拥有的各种各样的外形。 除了描述带有属性的普通对象外，接口也可以描述函数类型。
>
> 为了使用接口表示函数类型，我们需要给接口定义一个调用签名。 它就像是一个只有参数列表和返回值类型的函数定义。参数列表里的每个参数都需要名字和类型

```typescript
interface SearchFunction{
  (target:string):boolean
}
let search:SearchFunction = (target:string，substr:string) =>{
  return targer.indexof(substr) > -1;
}
```

- 可索引的接口类型

```typescript
interface StringArray {
  [index: number]: string;
}
let myArray: StringArray;
myArray = ["Bob", "Fred"];
```

*  接口继承类

> 当接口继承了一个类类型时，它会继承类的成员但不包括其实现。 就好像接口声明了所有类中存在的成员，但并没有提供具体实现一样。 接口同样会继承到类的private和protected成员。 这意味着当你创建了一个接口继承了一个拥有私有或受保护的成员的类时，这个接口类型只能被这个类或其子类所实现（implement）。

```typescript
class Control {
    private state: any;
}

interface SelectableControl extends Control {
    select(): void;
}

class Button extends Control implements SelectableControl {
    select() { }
}

class TextBox extends Control {
    select() { }
}

// 错误：“Image”类型缺少“state”属性。
class Image implements SelectableControl {
    select() { }
}
```

* extends 继承接口

> 和类一样，接口也可以相互继承。 这让我们能够从一个接口里复制成员到另一个接口里，可以更灵活地将接口分割到可重用的模块里。

```typescript
interface Shape {
    color: string;
}
interface PenStroke {
    penWidth: number;
}
interface Square extends Shape, PenStroke {
    sideLength: number;
}
let square = <Square>{};
square.color = "blue";
square.sideLength = 10;
square.penWidth = 5.0;
```

* 接口继承类

> 当接口继承了一个类类型时，它会继承类的成员但不包括其实现。 就好像接口声明了所有类中存在的成员，但并没有提供具体实现一样。 接口同样会继承到类的private和protected成员。 这意味着当你创建了一个接口继承了一个拥有私有或受保护的成员的类时，这个接口类型只能被这个类或其子类所实现（implement）。

```typescript
class Width{
    private width:number;
    constructor(width:number){
        this.width = width;
    }
    get(){
        return this.width;
    }
}
interface rectangle extends Width{
    height:number;
    getS():number
}
class Rectangle extends Width implements rectangle{
    height:number
    constructor(width:number,height:number){
        super(width)
        this.height = height
    }
    getS(){
        return this.get()*this.height;
    }
}
```

## 类类型

- 类实现接口

与C#或Java里接口的基本作用一样，TypeScript也能够用它来明确的强制一个类去符合某种契约。

```typescript
interface ClockInterface {
    currentTime: Date;
    setTime(d: Date);
}

class Clock implements ClockInterface {
    currentTime: Date;
    setTime(d: Date) {
        this.currentTime = d;
    }
    constructor(h: number, m: number) { }
}
```

## 高级类型

- keyof 关键字

入口索引类型查询或者说 keyof; 索引类型查询 keyof T 会得出 T 可能的属性名称的类型. keyof T 类型被认为是 string 的子类型.

Eg:

```typescript
interface Person {
    name: string;
    age: number;
    location: string;
}
type K1 = keyof Person; // "name" | "age" | "location"
type K2 = keyof Person[];  // "length" | "push" | "pop" | "concat" | ...
type K3 = keyof { [x: string]: Person };  // string
```

你可以将这种形式与类型系统中的其他功能组合, 来获得类型安全的查找.

```typescript
function getProperty<T, K extends keyof T>(obj: T, key: K) {
    return obj[key];  // 推断的类型为 T[K]
}

function setProperty<T, K extends keyof T>(obj: T, key: K, value: T[K]) {
    obj[key] = value;
}

let x = { foo: 10, bar: "hello!" };

let foo = getProperty(x, "foo"); // number
let bar = getProperty(x, "bar"); // string

let oops = getProperty(x, "wargarbl"); // 错误! "wargarbl" 不满足类型 "foo" | "bar"

setProperty(x, "foo", "string"); // 错误! string 应该是 number
```

K extends keyof  any ：支持任意传入的K 值