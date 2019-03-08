---
title: slot插槽的理解与使用
date: 2019-03-04 23:07:38
tags: slot
categories: [JavaScript,Vue.js]
---

> 最近在工作中用到了slot插槽,而以前在学习Vue的过程对插槽的使用比较少。所以趁着周末学习了一下插槽的相关内容，在这里做总结和记录。

<!-- more -->

​	在这里关于Vue.js中Vue的基本使用与组件的基础内容这里将不再讲解。如果想对Vue有基本的了解请移步[官方文档](https://cn.vuejs.org/v2/guide/components-slots.html)

### 什么是插槽

​	关于什么是插槽在[官方文档](https://cn.vuejs.org/v2/guide/components-slots.html)中的解释并不是非常的清晰。个人的理解是Vue中的插槽是一个占位符。当其父组件被使用时，可将插槽的内容替换。被替换的内容会出现在组件的对应位置。该内容可以使一段字符串，HTML模板或者是一个组件。

### 插槽内容

​	我们首先写一个插槽组件blog-post

```javascript
Vue.component('blog-post',{
    "template":`
		<div>
			<slot></slot>
		</div>
	`
})
```

​	我们可以在父组件中使用组件blog-post,在组件中用其他元素替换slot

```html
<blog-post>
    <div>this is slot text</div>
</blog-post>

```

​	查看结果我们可以看到组件blog-post中的插槽已经被书写的内容所占用

```html
<div>
    <div>
        this is slot text
    </div>
</div>
```



### 具名插槽

​	有时候我们会需要多个插槽，这时候我们需要给插槽定义一个名字，例如一个带有如下模板的**base-layout**组件

```html
<div id="container">
    <header>
    	...这里是页头
    </header>    
    <main>
    	...这里是主体部分
    </main>
    <footer>
    	...这里是页脚部分
    </footer>
</div>
```

​	面对这种情况，我们会使用不同的name去区分不同的插槽。一个未被取名的插槽默认name为default。创建一个模板**base-layout**

```javascript
Vue.component("base-layout",{
    template:`
		<div id="container">
			<header>
				<slot name="header"><slot>
			</header>
			<main>
				<slot></slot>
			</main>
			<footer>
				<slot name="footer"></slot>
			</footer>
		</div>
	`
})
```

​	在向具名插槽提供内容时，我们可以在``template``元素中使用v-slot:或者简写#指令（如v-on:与@对应一般）。且未提供v-slot的``template``都会被视为默认插槽的内容。我们可以写一个使用模板***base-layout***的例子

```html
<base-layout>
	<header>
    	<template v-slot="header">
        	<div>
                this is header text 
            </div>
        </template>
        <template> <!--为默认插槽不需要使用v-slot-->
        	<div>
                this is main text 
            </div>
        </template>
        <template #footer> <!-- #为v-slot 的缩写-->
        	<div>
                this is footer text
            </div>
        </template>
    </header>
</base-layout>
```

​	最终将渲染出结果

```html
<div id="container">
	<header>
        <div>
            this is header text 
        </div>			
    </header>
    <main>
        <div>
            this is main text 
        </div>			
    </main>
    <footer>
        <div>
            this is footer text
        </div>
    </footer>
</div>
```

### 作用域插槽

​	在插槽中较难理解的是作用域插槽，作用域插槽可以让父组件可以访问到子组件中的数据,作用域插槽在Vue中公共模板的抽离的作用是巨大的。作用域插槽与其函数写法scopedSlot的用法将在下一节讲解。

