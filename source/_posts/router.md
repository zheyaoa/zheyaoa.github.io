---
title: HashRouter和HistoryRouter的简单实现
date: 2019-12-11 15:48:26
tags: [router]
categories: [JavaScript]
---

最近在看 `vue-router` ，关于 `vue-router` 的一些分析可以看文章 [vue-router源码分析](https://zheyaoa.github.io/2019/11/29/vue-router-analyse/#more) ，我们们都知道 `vue-router` 拥有 `hash/history/abastract` 三种模式，其中 `abstract` 主要运用于 `SSR`, 本文主要讲述一些`hash/history`模式的简单实现

<!-- more -->

### HistoryRouter

说到 `History` 路由你就不得不提到 history 的 [pushState](https://developer.mozilla.org/zh-CN/docs/Web/API/History_API) 与 [replaceState](https://developer.mozilla.org/en-US/docs/Web/API/History/replaceState) 方法，这两个 API 都可以改变浏览器的路径而不进行实际的页面跳转,被大量用来实现各种 SPA 框架

`History`大致实现如下

```javascript
class HistoryRouter{
    constructor(){
      this._routes = {};
      this._history = window.history
      window.addEventListener('popstate',e => {
        console.log(e)
      })
    }
    addRoute(path,callback){
      this._routes[path] = callback
    }
    push(path){
      this._history.pushState({path},"",path)
      this._routes[path] && this._routes[path]()
    }
    replace(path){
      this._history.replaceState({path},"",path)
      this._routes[path] && this._routes[path]()
    }
}
```

`HistoryRouter` 暴露三个 API 负责 路由的添加与跳转

### HashRouter

`hash` 模式也是大家常用的一种模式了，它利用路由的 hash 来控制路由的跳转。 hash 模式相较于 history 模式更为复杂，大致实现如下

```
class HashRouter{
    constructor(){
        this.history = window.history
        this.support = !!this.history.pushState
        this.routes = {}
        window.location.hash = ''
        window.addEventListener(this.support?'popstate':'hashchange',e => {
            console.log(e)
        })
    }
    addRoute(path,callback){
        this.routes[path] = callback || function(){}
    }
    push(path){
        if(this.support){
            this.history.pushState({path},'',this.getUrl(path))
        }else{
            window.location.hash = path
        }
        this.routes[path]()
    }
    replace(path){
        if(this.support){
            this.history.replaceState({path},'',this.getUrl(path))
        }else{
            window.location.hash = path
        }
        this.routes[path]()
    }
    getUrl(path){
        return `#${path}`
    }
}
```

这里参考了 `vue-router`的实现，先去判断浏览器是否支持 `history.pushstate` ,如果支持则使用`pushState/replaceState` 否则使用 `window.location.hash` 进行 hash 的拼接

### Hash模式与History模式的一些问题

* History 模式丢掉了丑陋的 # ，比较美观
* 使用 Hash 模式容易与锚点混淆
* 使用 History 模式时因为使用的是真实的url,当你刷新时会向后端请求产生404