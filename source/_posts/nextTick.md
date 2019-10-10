---
title: 从源码角度分析函数nextTick
date: 2019-07-30 21:53:34
tags: nextTick
categories: [JavaScript,Vue.js]
---

最近在看vue.js 源码，大家都知道nextTick是vue.js中一个非常核心的部分。涉及到vuejs的异步DOM更新，所以着重的分析了一下，在这里做一个总结。

<!-- more -->

### 一个小Demo

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <script src="../../dist/vue.js"></script>
    <title>Document</title>
</head>
<body>
    <div id="app">
        <div ref='demo'>{{message}}</div>
        <button @click="handleClick">change</button>
    </div>
</body>
<script>
    new Vue({
        el:'#app',
        data(){
            return {
                message:'Old value'
            }
        },
        methods:{
            handleClick(){
                this.message = 'New Value'
                console.log(this.$refs['demo'].innerHTML)
              	this.$nextTick(()=>{
                  console.log(this.$refs['demo'].innerHTML)
                })
            }
        }
    })
</script>
</html>
```

**当点击change button点击后的输出**

![image-20190803105851684](http://cdn.zheyao.top/nextTick.png)

可以看到 `handleClick`函数的输出并没有如我们想象的一般（输出new value）而是输出了`old value`,这是为什么呢？

### 带着问题思考

我们知道 Vue 的响应式实现是依靠 `Object.definepropery` 去给Vue 的诸如data/props的响应式对象的key设定`get/set`，当对象被调用getter时，我们会做一些依赖的收集。调用set时，会使组件重新渲染。

而在 `handleClick ` 函数中，我们 执行逻辑`this.message = 'new Value'`,调用了对应的set函数，我们看一下`set` 函数的实现

```javascript
set: function reactiveSetter (newVal) {
  // 获取 oldValue  
  const value = getter ? getter.call(obj) : val
  /* eslint-disable no-self-compare */
  if (newVal === value || (newVal !== newVal && value !== value)) {
    return
  }
  /* eslint-enable no-self-compare */
  if (process.env.NODE_ENV !== 'production' && customSetter) {
    customSetter()
  }
  // #7981: for accessor properties without setter
  if (getter && !setter) return

  if (setter) {
    setter.call(obj, newVal)
  } else {
    val = newVal
  }
  // 如果说 newValue 也是一个对象,那么对 newValue 做一个响应式的处理
  childOb = !shallow && observe(newVal)
  dep.notify()
}
```

可以看到定义的set函数中，我们进行了一个进行了一些判断，获取了正确的值，最终走到了`dep.notify`

```javascript
notify () {
  // stabilize the subscriber list first
  // 做一份深拷贝
  const subs = this.subs.slice()
  if (process.env.NODE_ENV !== 'production' && !config.async) {
    // subs aren't sorted in scheduler if not running async
    // we need to sort them now to make sure they fire in correct
    // order
    subs.sort((a, b) => a.id - b.id)
  }
  for (let i = 0, l = subs.length; i < l; i++) {
    subs[i].update()
  } 
}
```

`notify`的定义也很简单，给Watche做一个简单排序，最终执行`sub.update`

那么`sub.update`是什么呢，我们可以简单将他理解为执行组件的渲染函数使对应的dom更新。

```javascript
update () {
  /* istanbul ignore else */
  if (this.lazy) {
    this.dirty = true
  } else if (this.sync) {
    this.run()
  } else {
    queueWatcher(this)
  }
}
```

在这里会执行`queueWatcher(this)`函数，将待执行函数（使dom重新渲染的函数）加入到一个队列中，等待一个合适的时机将待执行函数执行。

```javascript
export function queueWatcher (watcher: Watcher) {
  const id = watcher.id
  if (has[id] == null) {
    has[id] = true
    if (!flushing) {
      //还未进入 flushSchedulerQueue(flush status) 函数时
      queue.push(watcher)
    } else {
      // if already flushing, splice the watcher based on its id
      // if already past its id, it will be run next immediately.
      let i = queue.length - 1
      // index 是正在执行 flush(Watcher.run)函数的 Watcher 下标
      // Watcher 是当前
      while (i > index && queue[i].id > watcher.id) {
        i--
      }
      queue.splice(i + 1, 0, watcher)
    }
    // queue the flush
    if (!waiting) {
      waiting = true

      if (process.env.NODE_ENV !== 'production' && !config.async) {
        flushSchedulerQueue()
        return
      }
      nextTick(flushSchedulerQueue)
    }
  }
}
```

看到这里我们可能会了解，原来当我们执行`this.message = new Value `时，对应的渲染函数不会立即执行，而是会将渲染函数放入一个执行队列，然后调用`nextTick`调用执行队列,这样函数队列就会合适的时机执行。所以显然的当我们这时去执行`console.log(this.$refs['demo'].innerHTML)`,此时dom并没有被重新渲染。所以我们输出的值为`Old Value`

###  真正的核心 nextTick

上面我们了解了Vue.js 中 DOM 异步更新的问题，那本文真正的核心`nextTick`呢？

我们来看看nextTick的实现

```javascript
/* @flow */
/* globals MutationObserver */

import { noop } from 'shared/util'
import { handleError } from './error'
import { isIE, isIOS, isNative } from './env'

export let isUsingMicroTask = false

const callbacks = []
let pending = false

// 执行回调的逻辑
function flushCallbacks () {
  pending = false
  const copies = callbacks.slice(0)
  callbacks.length = 0
  for (let i = 0; i < copies.length; i++) {
    copies[i]()
  }
}

// Here we have async deferring wrappers using microtasks.
// In 2.5 we used (macro) tasks (in combination with microtasks).
// However, it has subtle problems when state is changed right before repaint
// (e.g. #6813, out-in transitions).
// Also, using (macro) tasks in event handler would cause some weird behaviors
// that cannot be circumvented (e.g. #7109, #7153, #7546, #7834, #8109).
// So we now use microtasks everywhere, again.
// A major drawback of this tradeoff is that there are some scenarios
// where microtasks have too high a priority and fire in between supposedly
// sequential events (e.g. #4521, #6690, which have workarounds)
// or even between bubbling of the same event (#6566).
let timerFunc

// The nextTick behavior leverages the microtask queue, which can be accessed
// via either native Promise.then or MutationObserver.
// MutationObserver has wider support, however it is seriously bugged in
// UIWebView in iOS >= 9.3.3 when triggered in touch event handlers. It
// completely stops working after triggering a few times... so, if native
// Promise is available, we will use it:
/* istanbul ignore next, $flow-disable-line */
if (typeof Promise !== 'undefined' && isNative(Promise)) {
  const p = Promise.resolve()
  timerFunc = () => {
    p.then(flushCallbacks)
    // In problematic UIWebViews, Promise.then doesn't completely break, but
    // it can get stuck in a weird state where callbacks are pushed into the
    // microtask queue but the queue isn't being flushed, until the browser
    // needs to do some other work, e.g. handle a timer. Therefore we can
    // "force" the microtask queue to be flushed by adding an empty timer.
    if (isIOS) setTimeout(noop)
  }
  isUsingMicroTask = true
} else if (!isIE && typeof MutationObserver !== 'undefined' && (
  isNative(MutationObserver) ||
  // PhantomJS and iOS 7.x
  MutationObserver.toString() === '[object MutationObserverConstructor]'
)) {
  // Use MutationObserver where native Promise is not available,
  // e.g. PhantomJS, iOS7, Android 4.4
  // (#6466 MutationObserver is unreliable in IE11)
  let counter = 1
  const observer = new MutationObserver(flushCallbacks)
  const textNode = document.createTextNode(String(counter))
  observer.observe(textNode, {
    characterData: true
  })
  timerFunc = () => {
    counter = (counter + 1) % 2
    textNode.data = String(counter)
  }
  isUsingMicroTask = true
} else if (typeof setImmediate !== 'undefined' && isNative(setImmediate)) {
  // Fallback to setImmediate.
  // Techinically it leverages the (macro) task queue,
  // but it is still a better choice than setTimeout.
  timerFunc = () => {
    setImmediate(flushCallbacks)
  }
} else {
  // Fallback to setTimeout.
  timerFunc = () => {
    setTimeout(flushCallbacks, 0)
  }
}

export function nextTick (cb?: Function, ctx?: Object) {
  let _resolve
  callbacks.push(() => {
    if (cb) {
      try {
        cb.call(ctx)
      } catch (e) {
        handleError(e, ctx, 'nextTick')
      }
    } else if (_resolve) {
      _resolve(ctx)
    }
  })
  if (!pending) {
    pending = true
    timerFunc()
  }
  // $flow-disable-line
  if (!cb && typeof Promise !== 'undefined') {
    return new Promise(resolve => {
      _resolve = resolve
    })
  }
}

```

nextTick 的原理其实也很简单，考虑到一些兼容性问题，他会按以下顺序寻找一个可用的异步函数`timerFunc`

1. Promise
2. MutationObserve
3. setimmediate
4. setTimeout

找到异步函数`timeFunc` 后用调用该函数，这样参数中的`callback`就会做到异步的调用了。

### 应用场景

我们已经了解到当我们去调用一个响应式对象的`set`时，依赖该响应式对象的dom不会及时的更新，而是会传到`nextTick`函数中到下一轮时间循环中被调用。

所以在数据变化后要执行的某个操作，而这个操作需要使用随数据改变而改变的DOM结构的时候，这个操作都应该放进`Vue.nextTick()`的回调函数中。

具体原因如下，来自Vue.js 官方文档

> 可能你还没有注意到，Vue 在更新 DOM 时是**异步**执行的。只要侦听到数据变化，Vue 将开启一个队列，并缓冲在同一事件循环中发生的所有数据变更。如果同一个 watcher 被多次触发，只会被推入到队列中一次。这种在缓冲时去除重复数据对于避免不必要的计算和 DOM 操作是非常重要的。然后，在下一个的事件循环“tick”中，Vue 刷新队列并执行实际 (已去重的) 工作。Vue 在内部对异步队列尝试使用原生的 `Promise.then`、`MutationObserver` 和 `setImmediate`，如果执行环境不支持，则会采用 `setTimeout(fn, 0)` 代替。
>
> 例如，当你设置 `vm.someData = 'new value'`，该组件不会立即重新渲染。当刷新队列时，组件会在下一个事件循环“tick”中更新。多数情况我们不需要关心这个过程，但是如果你想基于更新后的 DOM 状态来做点什么，这就可能会有些棘手。虽然 Vue.js 通常鼓励开发人员使用“数据驱动”的方式思考，避免直接接触 DOM，但是有时我们必须要这么做。为了在数据变化之后等待 Vue 完成更新 DOM，可以在数据变化之后立即使用 `Vue.nextTick(callback)`。这样回调函数将在 DOM 更新完成后被调用。