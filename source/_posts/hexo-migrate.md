---
title: hexo迁移
date: 2019-03-10 21:47:36
tags: hexo
categories: hexo
---

> 最近实习后公司配了Mac，所以以前自己的笔记本就用的比较少了（这货实在是太沉了 ）。电脑不用了，可是日常的博客还是得写，所以需要把hexo博客迁移到工作机上。在网上看的教程感觉有一点繁琐，这里提供一个比较方面的方法以供参考。

<!-- more -->

### 迁移环境的准备

​	在这里我们默认工作机与原来的电脑都已经装好了Git,node.js且都和Git仓库连接了SSH。如果未做好这步准备，可以参考[这篇文章](https://www.jianshu.com/p/ee2578821d49)。当环境已经准备好后，我们开始进行下一步。

### hexo迁移

​	当环境完全配置好后，我们可以开始我们hexo项目的迁移啦。

​	我们仔细查看一下我们的hexo目录

> Hexo
>
> > .deploy_git/
> >
> > _config.yml       
> >
> > db.json     
> >
> > node_modules/      
> >
> > package-lock.json 
> >
> > package.json   
> >
> > public/
> >
> > scaffolds/ 
> >
> >  source/         
> >
> >  themes/

​	事实上最简单粗暴的方法就是我们直接将这个项目压缩打包拷贝到另一台电脑上，但作为一个优秀的程序员。怎么能干这种事呢？事实上这也不利于博客在两台电脑上的同步。我们自然而然的就想到了在Git上去管理hexo目录下的文件了。我们在我们hexo博客的username.github.io中建立一个hexo分支去管理Hexo目录下的文件，master分支将存储静态页面。

​	git仓库的结构与功能如下

​	![image](http://cdn.zheyao.top/4906139-652af9cbae0a1a3d.png)

​	

### 将hexo目录迁移到github仓库下

​	接下来我们要做的就很简单了，我们只需让hexo目录与username.github.io建立联系即可。

1. 在命令行中将目录指向Blog的根目录
2. 如果使用了自定义主体如next,到next目录下删除.git 文件（不删除的话会造成待会git push失败）。如未使用则跳过这一步。
3. 重新回到Blog的根目录下，执行代码`git init`
4. `git checkout -b hexo `
5. `git romote add origin 你的项目地址` 这一步将与你远程仓库建立连接
6. `git add .`
7. `git commit -m"hexo migrate"`
8. `git push origin hexo`  

做完这一步后，你已经将本地的hexo目录提交到了Git 远程仓库，但是有一点值得注意的是。自定义主题是不会被上传到 Git 仓库的。为了统一博客的主题，当在新电脑中从远程仓库中拉取hexo目录时,需要重新配置博客的主题。这一点在后面会给出具体的解决办法。

### 新电脑中配置同步博客

​	当我们在新电脑中需要同步博客时，我们要做的也很简单。步骤如下

1. `git clone -b hexo 项目url`同步Git 仓库项目目录的hexo分支。
2. 切换到项目目录下
3. `npm install`
4. 在这里其实就已经可以运行博客啦。我们执行`hexo g \hexo s `就可以在**localhost:4000**上看到博客了。我们发现博客的样式发生了变化。
5. 这一点在上面已经讲过了，当我们push hexo目录时，自定义主题并不会提交到Git 仓库。我们只需要在新电脑中重新下载自定义主题的文件。然后用老电脑的自定义主题的配置文件将新下载的配置文件替换。就大功告成啦。

### 如何同步博客

​	当我们使用不同的电脑时，我们使用

```
git add .
git commit -m“update store”
git push
```

​	在新的电脑中

```
git pull 
```

这样就可以愉快的进行我们的创作了。



