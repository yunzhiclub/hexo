---
title: 计量项目前台部分Controller之间的关系
date: 2017-11-22 17:20:01
tags:
category: gaoliming
---

最简单的编程就是看着图编程。最难维护的项目就是一张图都没有的项目。
此文章简介一下自己在修改前台过程中整理的部分功能模块是整理的类图。

<!--more-->

## 类的继承之间的关系
![](/2017/11/22/instrument-audit/Class Diagram0.svg) 

左边这个分支是查看的功能，右边这个分支是审核的功能。Controller之间的关系基本就是这样的。

## V层和C层之间的图

![](/2017/11/22/instrument-audit/c_v.svg) 

我想这样更有助于我们理解整个的文件的结构。

## 简单的时序图

![](/2017/11/22/instrument-audit/c.svg) 

这个时序图主要是讲了一下用户在我的工作的时候点击查看按钮发生的事情，调用相应的方法获取获取哪个Controller。然后进获取相应的数据。这只是是一个很粗糙的博客，**想说的就是当我们在查看一个项目代码的时候如果自己不理解一定要自己画图记录代码的流程**。