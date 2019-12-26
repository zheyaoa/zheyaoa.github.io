---
title: optimize
date: 2019-12-09 20:08:16
tags: [nextTick]
categories: [front-end]
---

性能问题越来越成为前端火热的话题，因为随着项目的逐步变大，性能问题也逐步体现出来。为了提高用户的体验，减少加载时间，工程师们想尽一切办法去优化细节。

<!-- more -->

###网站开启的具体流程

我们先简述一下网站的开启到加载的步骤简述

1. 预处理
2. 查询DNS
3. 建立TCP连接
4. 发送请求
5. 等待响应
6. 接收数据（html传输）
7. 静态资源下载
8. 处理渲染(解析文档，执行`js/css`规则计算布局，渲染完成)





总结图片优化：怎样让图片加载得更快
一.图片压缩
   1.压缩png
         1)node-pngquant-native
         2)跨平台，压缩比高，压缩png24非常好
   2.压缩jpg
        1)jpegtran
        2)图片尺寸随着网络环境变化
二.不同网络环境下，加载不同的图片
三.响应式图片
      1.javascript 绑定时间检测窗口大小
      2.css媒体查询
         @media screen and (max-width:640px){ my_image{width:640px;}}
      3.img标签属性
      <img srcset="img-320w.jpg,img-640w.jpg 2x,img-960w.jpg 3x"scr="img-960w.jpg" alt="img" >
三.逐步加载图像
     1.使用同一占位符
     2.使用LQIP
     3.使用SQIP
四.真的图片吗？
     1.web font 代替图片
     2.使用data URI代替图片
     3 .采用Image spriting （雪碧图） 请求数就只有一个