---
title: Ajax/axios 请求的取消
date: 2019-07-26 13:55:54
tags: [ajax,axios]
categories: [JavaScript]
---

最近碰到一个需求是当页面（路由）进行跳转时，取消已经发送的ajax(axios)请求。关于如何做到在页面离开时触发函数 可以看看vue-router 的[官方文档](https://router.vuejs.org/zh/guide/advanced/navigation-guards.html)，在这里记录一下ajax/axios 取消请求的方式。

<!-- more -->

### 原始Xhr

原生xhr取消请求的方法很简单，就是调用xhr对象下的 abort 方法。

```javascript
const xhr = new XMLHttpRequest()
xhr.open("GET","https://api.github.com/");
xhr.send();
xhr.onreadystatechange=function(){
    if(xhr.readyState==4&&xhr.status==200){
        console.log(xhr.responseText);           
    }
}
xhr.abort()
```

### 使用axios

当大家使用例如vue之类的框架时，我们一般都是使用axios去做一些请求的发送的。

那么axios如何去取消请求的发送呢？ 我们找到了axios的[文档](https://www.kancloud.cn/yunye/axios/234845)

可以看到 axios 基于 *cancel token* 取消请求

```javascript
var CancelToken = axios.CancelToken;
var source = CancelToken.source();

axios.get('/user/12345', {
  cancelToken: source.token
}).catch(function(thrown) {
  if (axios.isCancel(thrown)) {
    console.log('Request canceled', thrown.message);
  } else {
    // 处理错误
  }
});

// 取消请求（message 参数是可选的）
source.cancel('Operation canceled by the user.');
```