---
title: Vue中computed的依赖收集
date: 2019-09-07 12:15:44
tags: Vue.js
categories: [JavaScript,Vue.js]
---

最近面试的时候遇到了一个碰到了一个题目，大意如下,Vue中是如何对一个computed做到依赖收集的？当时没能回答上来，面试结束后做了一个小小的梳理。在这里做一个总结

<!-- more -->

### 题目大意

题目大意如下,在Vue.js中如何对`this.a`做到依赖收集与更新派发的

```javascript
computed: {
 a() {
    return this.b + this.c;
  }
}
```

我们都知道在Vue.js中对data对象数据的依赖收集在`Object.defineProperty`的`get`方法中，具体实现大概如下

```javascript
 Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
      const value = getter ? getter.call(obj) : val
      //组件 渲染 组件
      if (Dep.target) {
        dep.depend()
        if (childOb) {
          childOb.dep.depend()
          if (Array.isArray(value)) {
            dependArray(value)
          }
        }
      }
      return value
    },
}
```

### Computed 的依赖收集

那Computed 是如何做到依赖收集的呢？还是看源码

```javascript
const computedWatcherOptions = { lazy: true }
function initComputed (vm: Component, computed: Object) {
  // $flow-disable-line
  const watchers = vm._computedWatchers = Object.create(null)
  // computed properties are just getters during SSR
  const isSSR = isServerRendering()
  for (const key in computed) {
    const userDef = computed[key]
    const getter = typeof userDef === 'function' ? userDef : userDef.get
    if (!isSSR) {
      watchers[key] = new Watcher(
        vm,
        getter || noop,
        noop,
        computedWatcherOptions
      )
    }
    if (!(key in vm)) {
      defineComputed(vm, key, userDef)
    }
  }
}
```

可以看到在`Vue `中，每一个 `computed` ，都是一个`Watcher` ,这就意味着它可以像**组件Watcher**一样，去做一些依赖收集的工作，在这里着重提一点`const computedWatcherOptions = { lazy: true }`,可以看到这个配置在创建Watcher时被导入，这个配置也是**computed**与**watch**的最大区别。

***我们可以将同一函数定义为一个方法而不是一个计算属性。两种方式的最终结果确实是完全相同的。然而，不同的是计算属性是基于它们的响应式依赖进行缓存的。只在相关响应式依赖发生改变时它们才会重新求值。***

Watcher的代码实现

```javascript
export default class Watcher {
  vm: Component;
  expression: string;
  cb: Function;
  id: number;
  deep: boolean;
  user: boolean;
  lazy: boolean;
  sync: boolean;
  dirty: boolean;
  active: boolean;
  deps: Array<Dep>;
  newDeps: Array<Dep>;
  depIds: SimpleSet;
  newDepIds: SimpleSet;
  before: ?Function;
  getter: Function;
  value: any;

  constructor (
    vm: Component,
    expOrFn: string | Function,
    cb: Function,
    options?: ?Object,
    isRenderWatcher?: boolean
  ) {
    this.vm = vm
    if (isRenderWatcher) {
      vm._watcher = this
    }
    vm._watchers.push(this)
    // options
    if (options) {
      this.deep = !!options.deep
      this.user = !!options.user
      this.lazy = !!options.lazy
      this.sync = !!options.sync
      this.before = options.before
    } else {
      this.deep = this.user = this.lazy = this.sync = false
    }
    this.cb = cb
    this.id = ++uid // uid for batching
    this.active = true
    this.dirty = this.lazy // for lazy watchers
    this.deps = []
    this.newDeps = []
    this.depIds = new Set()
    this.newDepIds = new Set()
    // parse expression for getter
    if (typeof expOrFn === 'function') {
      this.getter = expOrFn
    }
    this.value = this.lazy
      ? undefined
      : this.get()
  }
```

可以看到 实现的末尾 `    this.value = this.lazy? undefined: this.get()`,因为此时的`option`中`lazy:true`实际上值`undefined`

### 回到问题

好了说了那么多，我们现在来回到问题。

一个简单的Demo

```html
<body>
    <div id="app">
        <div>{{a}} </div>
        <button @click=handleClick>click</button>
    </div>
</body>
<script>
    var vm = new Vue({
        el:'#app',
        methods:{
          handleClick(){
            this.message++
          }
        },
        computed:{
          a(){
            return this.message;
          }
        },
        data:()=>{
            return {
                message: 1,
            }
        }
    })
</script>
```

我们知道当我们去初始化组件的时候，此时的`Dep.target`指向该组件所在的`Watcher`,我在这就称它为`Wapp`,在渲染该组件时，会调用`this.a`,那么就会调用`this.a`的`get`函数

```javascript
export function defineComputed (
  target: any,
  key: string,
  userDef: Object | Function
) {
  const shouldCache = !isServerRendering()
  if (typeof userDef === 'function') {
    sharedPropertyDefinition.get = shouldCache
      ? createComputedGetter(key)
      : createGetterInvoker(userDef)
    sharedPropertyDefinition.set = noop
  }
  // 将 方法 挂载 在 vm 实例上
  Object.defineProperty(target, key, sharedPropertyDefinition)
}
function createComputedGetter (key) {
  return function computedGetter () {
    const watcher = this._computedWatchers && this._computedWatchers[key]
    if (watcher) {
      // 如果 dirty == true ,即代表 key 的 getter 函数被调用过
      // 则重新计算  否则不进行计算
      if (watcher.dirty) {
        // 在这里调用watcher 的 get 函数 Dep.target 指向 watcher
        // deps 在这里被修改
        watcher.evaluate()
      }
      // 依赖收集阶段 Target 为一个 Watcher
      if (Dep.target) {
        // 让 watcher 订阅 deps 中的 dep
        watcher.depend()
      }
      return watcher.value
    }
  }
}
```

记住,此时的`Dep.target`依旧指向`Wapp`，调用`get`时,`dirty`初始值为`true`,调用`watcher.evaluate()`

```javascript
evaluate () {
  this.value = this.get()
  this.dirty = false
}
get () {
  pushTarget(this)
  let value
  const vm = this.vm
  value = this.getter.call(vm, vm)
  if (this.deep) {
  traverse(value)
  }
  popTarget()
  this.cleanupDeps()
  }
  return value
}
```

最终会执行`watcher.get`,此时`  pushTarget(this)`,`Dep.target`指向`Computed`,我们称他为`CW`,然后执行`value = this.getter.call(vm, vm)`，即例子中的`return this.message;`,调用了`message`的`get`函数，那么`message`就会收集到`CW`的依赖。最后`popTarget`,此时`Dep.target`又变为`WA`

此时，整个`evaluate`执行结束

```javascript
if (watcher.dirty) {
  // 在这里调用watcher 的 get 函数 Dep.target 指向 watcher
  // deps 在这里被修改
  watcher.evaluate()
}
// 依赖收集阶段 Target 为一个 Watcher
if (Dep.target) {
  // 此时的Watcher 为 组件 Watcher
  // 让 watcher 订阅 deps 中的 dep
  watcher.depend()
}
```

从上文我们可以分析到，在执行计算Watcher的get函数时，已经收集到了一些依赖。（`this.deps = [Dep(a),Dep(b)]`）,此时执行`watcher.depend()`

```javascript
class Watcher{
    depend () {
    let i = this.deps.length
    while (i--) {
      //deps 是被 当前 Watcher 订阅的 发布者
      this.deps[i].depend()
    }
  }
}
class Dep{
  depend () {
    if (Dep.target) {
      Dep.target.addDep(this)
    }
  }
}
```

此时让`计算Watcher`的收集的`Dep`再次执行`depend`,注意此时的`Dep.target`指向`全局Watcher`,则`Dep(a),Dep(b)`收集到了收集到了`全局Watcher`的依赖。

至此，`计算Watcher`的依赖收集完成

### 计算Watcher 的派发更新

在进行完上面一系列操作之后,我们可以大概得到一个这样的依赖收集情况。

> Dep(a):[计算Watcher,全局Watcher]
>
> Dep(b):[计算Watcher,全局Watcher]

当我们的`this.a`或者`this.b`发生变化时。对应的`Dep`执执行`notify`,通知各个`Watcher`执行`callback`。

1. **计算Watcher** 执行 callback,使 `this.dirty = true`

2. **全局Watcher** 执行 callback,使得浏览器重新渲染组件。因为`计算Watcher`的`dirty`为`true`,会被正确更新。



至此，computed的依赖收集，派发更新分析结束

