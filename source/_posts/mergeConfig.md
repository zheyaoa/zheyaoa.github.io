---
title: Vue.js的配置合并
date: 2019-11-12 15:10:42
tags: Vue.js
categories: [JavaScript,Vue.js]
---

Vue.js 的配置合并主要有两种场景，一种是我们直接调用Vue.js的`Vue.extend,Vue.component,Vue.mixin`等钩子时， 另一种是使用new Vue实例化一个Vue 实例或者一个Vue 组件实例时，这里对两种情况做一些分析。

<!-- more -->

### 调用Vue.mixin/Vue.extend 时

***Vue.mixin/Vue.extend*** 都是Vue 内部提供的一些[全局API](https://cn.vuejs.org/v2/api/#Vue-component),通过这些全局钩子我们可以对Vue.js 配置做一写全局的混入。 在这里举一个具体实例

```javascript
<div id="app"></div>
<script>
      Vue.mixin({
        beforeCreate(){
          console.log("parent:before-created hooks")
        }
      })
      const vm = new Vue({
        el:"#app",
        beforeCreate(){
          console.log("son:before-created hooks")
        }
      })
</script>
```

我们先调用了`Vue.mixin`函数对`Vue.js`配置做一个混入，然后成功实例化了一个`Vue.js`实例。执行结果如下

![image-20191112155015956](/Users/yuxin.yu/Library/Application Support/typora-user-images/image-20191112155015956.png)

我们对`Vue.mixin`实现进行分析，`Vue.mixin`的实现在`src/core/global-api/mixin.js`中，`Vue.mixin`的实现就是将`this.options`与传入的对象`mixinOption`做了一层mergeOptions。即`this.options = mergeOptions(this.options,mixin)`。所以`Vue.js` 配置合并的核心即为函数`mergerOptions`

```js
import { mergeOptions } from '../util/index'
export function initMixin (Vue: GlobalAPI) {
  Vue.mixin = function (mixin: Object) {
    this.options = mergeOptions(this.options, mixin)
    return this
  }
}
```

### 实例化一个Vue实例时

同样的，当我们实例化一个`Vue/VueComponent`，我们同样做同样需要做一些配置合并的工作

```javascript
if (options && options._isComponent) {
  initInternalComponent(vm, options)
} else {
  vm.$options = mergeOptions(
    resolveConstructorOptions(vm.constructor),
    options || {},
    vm
  )
}
```

当`Vue`实例是一个组件实例时，会走向`if`的逻辑，如果是一个`Vue` 实例时，会走向`else`的逻辑

先看看`else`的逻辑,同样举一个具体实例加以说明

```javascript
</head>
<body>
    <div id="app">
      <user name="yyx"></user>
    </div>
    <script>
      const user = Vue.extend({
        name:'user',
        props:{
          name:String
        },
        template:`
          <div> hello world {{this.name}}</div>
        `
      })
      const vm = new Vue({
        el:"#app",
        components:{
          user
        },
        props:{
          age:Number
        },
        beforeCreate(){
          console.log("son:before-created hooks")
        }
      })
    </script>
</body>
```

首先执行`Vue.extend`创造了一个`VueComponent`,`Vue.extend`的实现在`src/core/global-api/extend.js`

```javascript
Vue.extend = function (extendOptions: Object): Function {
    extendOptions = extendOptions || {}
    const Super = this
    const SuperId = Super.cid
    const cachedCtors = extendOptions._Ctor || (extendOptions._Ctor = {})
    if (cachedCtors[SuperId]) {
      return cachedCtors[SuperId]
    }
    const name = extendOptions.name || Super.options.name
    const Sub = function VueComponent (options) {
      this._init(options)
    }
    Sub.prototype = Object.create(Super.prototype)
    Sub.prototype.constructor = Sub
    Sub.cid = cid++
    Sub.options = mergeOptions(
      Super.options,
      extendOptions
    )
    Sub['super'] = Super
    if (Sub.options.props) {
      initProps(Sub)
    }
    if (Sub.options.computed) {
      initComputed(Sub)
    }
    Sub.superOptions = Super.options
    Sub.extendOptions = extendOptions
    Sub.sealedOptions = extend({}, Sub.options)
    cachedCtors[SuperId] = Sub
    return Sub
  }
}
```

`Vue.extend`的逻辑也很清晰，创造一个新的构造函数`Sub`,利用`Sub.prototype = Object.create(Super.prototype)`使`Sub`继承`Super`的属性，然后使用`this.options = mergeOptions(Super.options,extendOption)`利用一定的规则将`Super.options`与`extendOptions`混合。最后返回一个`VueComponent`的构造函数，在例子中，当代码实例化`<user name="yyx"></user>`组件会调用他的构造函数最终走到

```javascript
  vm.$options = mergeOptions(
    resolveConstructorOptions(vm.constructor),
    options || {},
    vm
  )
```

### mergeOptions 的简单分析

上面我们分析了几种情况，发现代码最后都走向了`mergeOptions`，我们来简单分析一下`mergeOptions`的实现

`mergeOptions`的实现在`src/core/util/option.js`处

```javascript
export function mergeOptions (
  parent: Object,
  child: Object,
  vm?: Component
): Object {
  const options = {}
  let key
  for (key in parent) {
    mergeField(key)
  }
  for (key in child) {
    if (!hasOwn(parent, key)) {
      mergeField(key)
    }
  }
  function mergeField (key) {
    const strat = strats[key] || defaultStrat
    options[key] = strat(parent[key], child[key], vm, key)
  }
  return options
}
```

`mergeOption`的核心逻辑如上，对`parent`与`child`上的每一个`key`调用`mergeField`函数，`Vue.js`对不同的属性有不同`merge strats`。

#### componets 的合并策略

`Vue.js`对`components`的合并策略如下

```javascript
function mergeAssets (
  parentVal: ?Object,
  childVal: ?Object,
  vm?: Component,
  key: string
): Object {
  const res = Object.create(parentVal || null)
  if (childVal) {
    process.env.NODE_ENV !== 'production' && assertObjectType(key, childVal, vm)
    debugger
    return extend(res, childVal)
  } else {
    return res
  }
}
```

逻辑也很简单，将`parentVal`的属性放在`res`的原型上，再将`res`与`childVal`做一层合并

#### LIFECYCLE_HOOKS（生命周期钩子）的合并策略

```javascript
function mergeHook (
  parentVal: ?Array<Function>,
  childVal: ?Function | ?Array<Function>
): ?Array<Function> {
  const res = childVal
    ? parentVal
      ? parentVal.concat(childVal)
      : Array.isArray(childVal)
        ? childVal
        : [childVal]
    : parentVal
  return res
    ? dedupeHooks(res)
    : res
}
```

### 实例化Vue 组件的合并策略

最后没有分析到的是组件实例的合并策略，即

```javascript
if (options && options._isComponent) {
  initInternalComponent(vm, options)
}
```

在何种情况`options._isComponent` 为 `true` 呢,

```javascript
export function createComponentInstanceForVnode (
  vnode: any, // we know it's MountedComponentVNode but flow doesn't
  parent: any, // activeInstance in lifecycle state
): Component {
  const options: InternalComponentOptions = {
    _isComponent: true,
    _parentVnode: vnode, // 父vnode 为占位符 vnode ,将生成一个真正的vnode
    parent //父实例
  }
  // check inline-template render functions
  // 函数组件
  const inlineTemplate = vnode.data.inlineTemplate
  if (isDef(inlineTemplate)) {
    options.render = inlineTemplate.render
    options.staticRenderFns = inlineTemplate.staticRenderFns
  }
  // 返回了一个 组件实例
  return new vnode.componentOptions.Ctor(options)
}
```

此时`options._isComponent`为`true`走`if`逻辑

```javascript
  const opts = vm.$options = Object.create(vm.constructor.options)
  // doing this because it's faster than dynamic enumeration.
  const parentVnode = options._parentVnode
  opts.parent = options.parent
  opts._parentVnode = parentVnode

  const vnodeComponentOptions = parentVnode.componentOptions
  opts.propsData = vnodeComponentOptions.propsData
  opts._parentListeners = vnodeComponentOptions.listeners
  opts._renderChildren = vnodeComponentOptions.children
  opts._componentTag = vnodeComponentOptions.tag

  if (options.render) {
    opts.render = options.render
    opts.staticRenderFns = options.staticRenderFns
  }
```

逻辑其实也很简单，首先继承构造器的属性,虽然将一些`VueComponent`上的属性挂载在实例的`$options`上

