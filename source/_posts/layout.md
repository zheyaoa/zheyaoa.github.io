---
title: 详谈常见的圣杯/双飞翼布局
date: 2019-07-18 21:50:01
tags: layout
categories: CSS
---

我，前端菜鸡。今天面百度的时候被问到了所谓的圣杯，双飞翼布局。这两个东西一直都有所了解但是没真正的研究过，所以今天脸都被打肿了！所以今天仔细研究了一下所谓的双飞翼/圣杯布局在这里做一个总结。

<!-- more -->

所谓的双飞燕/圣杯布局,就是实现所谓的两边定宽，中间宽度随着浏览器的宽度响应式变化。两者的实现方法有略微的不同，这里分别进行分析。

### 圣杯布局

圣杯布局的实现可以简单理解为：

* 给container设置对应div宽度的padding,三个子div设置`postion:relative`
* 左边div设置`margin-left:-100%;left:-左边div宽度`
* 右边div设置`margin-left:-右边div宽度;right:右边div宽度`

代码如下

css 样式如下:

```css
<style>
html,body{
  height: 100%;
  width: 100%
}
.header{
  height: 100px;
  background: red
}
.container{
  height: 200px;
  padding-left: 200px;
  padding-right: 150px;
  box-sizing: border-box;
}
.container div{
  float: left;
  position: relative;
  height: 200px;
}
.left{
  width: 200px;
  margin-left: -100%;
  left: -200px;
  background: blue
}
.main{
  width: 100%;
  background: pink
}
.right{
  width: 150px;
  margin-left: -150px;
  left: 150px;
  background: yellow
}
.footer{
  clear: both;
  height: 100px;
  background: green
}
</style>
```

HTML布局

```html
<body>
    <div class="header"></div>
    <div class="container">
        <div class="main"></div>
        <div class="left"></div>
        <div class="right"></div>
    </div>
    <div class="footer"></div>
</body>
```

效果如图所示

![](http://cdn.zheyao.top/holyGrail.png)



* 双飞燕布局

双飞燕的实现还是与圣杯布局挺类似的，这里不做过多解释看代码就能了解了。

css代码如下

```css
<style>
        .header{
            background: red;
            height: 35px;
        }
        .container{
            height: 100px;
        }
        .container>div{
            float: left;
            height: 100px;
        }
        .main{
            height: 100%;
            width: 100%;
        }
        .inner{
            height: 100%;
            background: yellow;
            margin: 0 150px 0 200px;
        }
        .left{
            height: 100px;
            width: 200px;
            background: blue;
            margin-left: -100%;
        }
        .right{
            height: 100px;
            width: 150px;
            background: green;
            margin-left: -150px;
        }
        .footer{
            height: 35px;
            background: pink
        }
</style>
```

html 布局

```html
<body>
    <div class="header"></div>
    <div class="container">
        <div class="main">
            <div class="inner"></div>
        </div>
        <div class="left"></div>
        <div class="right"></div>
    </div>
    <div class="footer"></div>
</body>
```

这里的关键是***main***中有一个***inner***，***inner*** 去控制我们所需要的中间列的宽度,marin空出的长度留给left,right.

效果如下

![](http://cdn.zheyao.top/fly.png)

