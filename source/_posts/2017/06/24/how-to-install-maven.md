---
title: 如何安装maven
date: 2017-06-24 11:52:39
tags: [maven,install]
category: zhangjiahao
---
# 问题描述
在做单元测试以及对项目文件进行打包的时候，我们需要用到mvntest、mvnpackage等命令，这样做的前提是我们的电脑中必须安装好了maven，那么maven应该如何安装呢？
<!-- more -->
# 安装过程
## 下载maven
首先我们应该去maven的下载官网：[http://maven.apache.org/download.cgi](http://maven.apache.org/download.cgi)，找到如下图所示的文件进行下载。

![](/images/downloadmaven.png)

## 解压
下载完毕后进行解压，将maven放到我们想放到的磁盘下面，打开文件发现里面有一个bin文件。例如笔者的bin文件放在了G:\maven\apache-maven-3.5.0下面，如下图所示：

![](/images/locationMaven.png)

## 配置系统变量
接下来我们需要配置环境变量：控制面板->系统和安全->系统->高级系统设置->系统变量：

![](/images/environmentalParam.png)

在系统变量下点击新建，输入系统变量的值(由maven安装位置决定)

![](/images/systemEnvironment.png)

修改path值

![](/images/path.png)

## 测试是否安装成功
打开cmd，输入mvn --version

![](/images/maven.png)

如果出现这样的界面，恭喜你，maven已经成功安装到计算机中啦！
