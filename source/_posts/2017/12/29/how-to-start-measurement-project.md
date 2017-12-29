---
title: 如何启动测量项目
date: 2017-12-29 14:08:37
tags: [gateway, SpringMvc]
categories: chuhang
---

作为程序开发人员，启动项目是开发的第一步。那么，我们如何才能启动测量项目呢，为什么要这样设计。本篇文章为您简单介绍一下。
<!--more--> 

## 如何启动测量项目
1.拉取最新的开发分支，分别启动``GatewayApplication``、``ResourceApplication``
![](/2017/12/29/how-to-start-measurement-project/1.png) 
2.切换到Webapp目录下，``grunt serve``启动前台。
3.访问： http://localhost:8080
> 如果不能正常启动项目，建议idea重新导入项目

## 原因说明
我们可以观察到，当我们``grunt serve``启动前台时，实际上启动的是8083端口，为什么通过8080端口才能访问前台呢？我们访问http://localhost:8080 ，实际上是访问``GatewayApplication``应用程序，``Gateway``想当于代理的作用，我们观察``Gateway``的配置信息。
```
ribbon.eureka.enabled=false
# 资源地址
zuul.routes.resource.path=/resource/**
# 不排除任何header信息
zuul.routes.resource.sensitiveHeaders=
zuul.routes.resource.url=${resource.url:http://localhost:8081}

# 认证地址(获取accessToken)
zuul.routes.oauthToken.path=/oauth/token
# 携带前缀转发数据
zuul.routes.oauthToken.stripPrefix=false
zuul.routes.oauthToken.sensitiveHeaders=
zuul.routes.oauthToken.url=${oauth.url:http://localhost:8082}

# 认证地址(其它)
zuul.routes.oauth.path=/oauth/**
# 携带前缀转发数据
zuul.routes.oauth.url=${oauth.url:http://localhost:8082}

# 其它地址
zuul.routes.ui.path=/**
zuul.routes.ui.url=${ui.url:http://localhost:8083}

server.port=${port:8080}
```
1.首先我们可以从以下代码中看出，``Gateway``的端口是8080
```
server.port=${port:8080}
```

2.而当我们访问http://localhost:8080 时，``Gateway``把路由转发到了8083端口，所以就访问到了前台界面。代码如下：
```
zuul.routes.ui.path=/**
zuul.routes.ui.url=${ui.url:http://localhost:8083}
```

3.登陆前台界面，打开网络选项卡
![](/2017/12/29/how-to-start-measurement-project/2.png) 
我们可以看到，后台的api信息是以``http://localhost:8080/resource/``开头的，代码如下：
```
zuul.routes.resource.path=/resource/**
zuul.routes.resource.url=${resource.url:http://localhost:8081}
```
这样，当我们访问的路由是以/resource/开头时，就转发到了8081端口。

## 总结
通过Gateway应用程序，即使在前后台分离的情况下，也可以避免跨域的问题，即：协议、域名、端口号都是一样的。从而提高了用户的安全指数。