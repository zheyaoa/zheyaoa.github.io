---
title: Vue-router 源码分析
date: 2019-11-29 12:21:36
tags: [vue-router]
categories: [Vue.js] 
---

对于以Vue.js 为主要技术栈的前端工作者来说，熟练的使用Vue-router也是必须的。要想熟练的使用某个工具,必须了解其内部实现的原理。这里对vue-router的实现流程做一个梳理。

<!-- more -->

### vue-router 的安装

`vue-route`本质上是一个基于 `vue.js` 的工具，在`vue.js`中使用一些自定义的工具一般通过`Vue.use`将工具挂载

```javascript
import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter)
```

当执行这段代码后，会执行`vue-router/src/install.js`下的`install`函数

```javascript
export function install (Vue) {
  if (install.installed && _Vue === Vue) return
  install.installed = true
  const isDef = v => v !== undefined
  const registerInstance = (vm, callVal) => {
    let i = vm.$options._parentVnode
    if (isDef(i) && isDef(i = i.data) && isDef(i = i.registerRouteInstance)) {
      i(vm, callVal)
    }
  }
  Vue.mixin({
    beforeCreate () {
      if (isDef(this.$options.router)) {
        this._routerRoot = this
        this._router = this.$options.router
        this._router.init(this)
        Vue.util.defineReactive(this, '_route', this._router.history.current)
      } else {
        this._routerRoot = (this.$parent && this.$parent._routerRoot) || this
      }
      registerInstance(this, this)
    },
    destroyed () {
      registerInstance(this)
    }
  })
  Object.defineProperty(Vue.prototype, '$router', {
    get () { return this._routerRoot._router }
  })
  Object.defineProperty(Vue.prototype, '$route', {
    get () { return this._routerRoot._route }
  })
  Vue.component('RouterView', View)
  Vue.component('RouterLink', Link)
  const strats = Vue.config.optionMergeStrategies
  // use the same hook merging strategy for route hooks
  strats.beforeRouteEnter = strats.beforeRouteLeave = strats.beforeRouteUpdate = 	strats.created
}
```

`install`的逻辑很简单，大体流程如下。

1. 判断是否注册过`vue-router`组件，若注册过，则退出。否则开始注册
2. 利用`Vue.mixin`在每一个`vue`实例中挂载`beforeCreate hooks` 
3. 在`vue`实例上挂载`$router/$route`属性
4. 注册`router-view/router-link`组件

### vue-router实例的创建

`vue-router`的使用通过`new Vue(options)`的方式实现初始化，代码位于`vue-router/src/index.js`

```javascript
export default class VueRouter {
  static install: () => void;
  static version: string;

  app: any;
  apps: Array<any>;
  ready: boolean;
  readyCbs: Array<Function>;
  options: RouterOptions;
  mode: string;
  history: HashHistory | HTML5History | AbstractHistory;
  matcher: Matcher;
  fallback: boolean;
  beforeHooks: Array<?NavigationGuard>;
  resolveHooks: Array<?NavigationGuard>;
  afterHooks: Array<?AfterNavigationHook>;

  constructor (options: RouterOptions = {}) {
    this.app = null
    this.apps = []
    this.options = options
    this.beforeHooks = []
    this.resolveHooks = []
    this.afterHooks = []
    this.matcher = createMatcher(options.routes || [], this)

    let mode = options.mode || 'hash'
    this.fallback = mode === 'history' && !supportsPushState && options.fallback !== false
    if (this.fallback) {
      mode = 'hash'
    }
    if (!inBrowser) {
      mode = 'abstract'
    }
    this.mode = mode
    switch (mode) {
      case 'history':
        this.history = new HTML5History(this, options.base)
        break
      case 'hash':
        this.history = new HashHistory(this, options.base, this.fallback)
        break
      case 'abstract':
        this.history = new AbstractHistory(this, options.base)
        break
      default:
        if (process.env.NODE_ENV !== 'production') {
          assert(false, `invalid mode: ${mode}`)
        }
    }
  }
}
```

#### 创建matcher-map

初始化`vue-router`实例时首先会根据`option`创建对应的`matcher-map`,代码实现在

```javascript
this.matcher = createMatcher(options.routes || [], this)    
```

`createMatcher`的大致流程如下

```javascript
export function createMatcher (
  routes: Array<RouteConfig>,
  router: VueRouter
): Matcher {
  const { pathList, pathMap, nameMap } = createRouteMap(routes)
  function addRoutes (routes) {
    createRouteMap(routes, pathList, pathMap, nameMap)
  }
  function match (
    raw: RawLocation,
    currentRoute?: Route,
    redirectedFrom?: Location
  ): Route {
    const location = normalizeLocation(raw, currentRoute, false, router)
    const { name } = location
    if (name) {
      const record = nameMap[name]
      if (process.env.NODE_ENV !== 'production') {
        warn(record, `Route with name '${name}' does not exist`)
      }
      if (!record) return _createRoute(null, location)
      const paramNames = record.regex.keys
        .filter(key => !key.optional)
        .map(key => key.name)

      if (typeof location.params !== 'object') {
        location.params = {}
      }

      if (currentRoute && typeof currentRoute.params === 'object') {
        for (const key in currentRoute.params) {
          if (!(key in location.params) && paramNames.indexOf(key) > -1) {
            location.params[key] = currentRoute.params[key]
          }
        }
      }

      location.path = fillParams(record.path, location.params, `named route "${name}"`)
      return _createRoute(record, location, redirectedFrom)
    } else if (location.path) {
      location.params = {}
      for (let i = 0; i < pathList.length; i++) {
        const path = pathList[i]
        const record = pathMap[path]
        if (matchRoute(record.regex, location.path, location.params)) {
          return _createRoute(record, location, redirectedFrom)
        }
      }
    }
    // no match
    return _createRoute(null, location)
  }
  function _createRoute (
    record: ?RouteRecord,
    location: Location,
    redirectedFrom?: Location
  ): Route {
    if (record && record.redirect) {
      return redirect(record, redirectedFrom || location)
    }
    if (record && record.matchAs) {
      return alias(record, location, record.matchAs)
    }
    return createRoute(record, location, redirectedFrom, router)
  }
  return {
    match,
    addRoutes
  }
}
```

`createMatcher`首先将执行`createRouteMap(routes)`函数，将对`option`生成对应的`pathList,pathMap与nameMap`,然后暴露`matcher 与 addRoutes`两个函数。

```
function match (
    raw: RawLocation,
    currentRoute?: Route,
    redirectedFrom?: Location
 ): Route 
```

`matcher`传入当前路径`row`与相对路径`currentRoute`获得一个新的路径，然后在`pathList/nameList`中返回对应的`router`

```javascript
function addRoutes (routes) {
	createRouteMap(routes, pathList, pathMap, nameMap)
}
```

`addRoutes`的逻辑也很简单，往`pathList,pathMap,nameMap`添加新的路由信息，同时`vue-router`暴露了一个`api:addRoutes`就是调用了这个方法。

