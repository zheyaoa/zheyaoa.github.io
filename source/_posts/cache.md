---
title: 浏览器缓存机制
date: 2019-12-08 23:10:09
tags: [cache,性能优化]
---

缓存是将浏览器请求的静态资源`html/css/js`,储存在本地磁盘中，当浏览器去请求同样的资源时，就可以直接从本地加载,不需要再次去请求了。

浏览器缓存是浏览器性能优化一种简单高效的方法

<!-- more -->

### 浏览器缓存过程分析

##缓存策略

###强缓存

> 强缓存可以用使用两种 `Expires`和`Cache-Control`，强缓存表示在缓存期间不需要发送请求，状态码为200 

#### Expires

```
Expires: Wed, 22 Oct 2018 08:41:00 GMT
```

`Expires`是`HTTP1.0`的产物，表示资源会在时间`Wed, 22 Oct 2018 08:41:00 GMT`后过期，需要再次请求。`Expires`基于计算机本地时间，当本地时间被修改，缓存可能会失效

#### Cache-Control

```Cache-control: max-age=30```

`Cache-Control`是`HTTP1.1`的产物，他的优先级高于`Expires`,上文header头表示缓存在30s后过期，需要再次请求。

Cache-Control 可能的header头如下

1. public: 表示响应可以被客户端和代理服务器缓存
2. private:表示响应只能被客户端缓存
3. max-age=30:缓存30s后过期,需要重新请求
4. s-maxage=30:覆盖max-age 只在代理服务器中生效
5. no-store:不缓存任何响应
6. no-cache:资源被缓存，但立即失效，下次请求会验证是否过期
7. max-stale:30s内，即使缓存过期,也是用该缓存

### 协商缓存

> 如果缓存过期了，就需要重新发起验证资源是否更新。协商缓存可以通过设置两种HTTP Header实现：`Last-Modifed`和`Etag`

当强缓存过期后，浏览器会发起请求验证资源。

当浏览器发起请求验证资源时，如果资源没有做改变，那么服务端就会返回 304 状态码，并且更新浏览器缓存有效期。

#### Last-Modified 和 If-Modified-Since

`Last-Modified`表示本地文件最后修改日期，`If-Modified-Since`会将`Last-Modified`的值发送给服务器，询问服务器在该日期后资源是否有更新，如果有更新就将新更新的资源发送回来，否则返回`304`状态码

Last-Modified 的缺陷

1. 如果在本地打开缓存文件，即使没有对文件进行修改，还是会造成`Last-Modified`被修改，服务端不能命中缓存导致发送相同的资源
2. Last-Modified是以秒计时的，如果以不可感知的时间修改了缓存文件。那么服务端会认为缓存还是命中了，不会返回正确的资源。

#### Etag 和 If-None-Match

`Etag`类似于文件指纹，`If-None-Match`会将当前`Etag`发送给服务器，询问该资源`Etag`是否变动，有变动的话就将新的资源发送回来。并且`Eatg`优先级比`Last-Modified`高hu'y