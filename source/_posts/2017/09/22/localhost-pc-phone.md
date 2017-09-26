---
title: 旅游项目如何实现手机预览
date: 2017-09-22 14:28:55
tags:
category: gaoliming
---

当我们真的想实现手机微信扫一扫实现预览网页的功能的时候我们不必将项目放在服务器上，本地也可以。

<!--more-->

**环境** : ubuntu16.04，android手机

## ifconfig

首先我们在终端输入``ifconfig``来查看本机的ip地址。
![](/2017/09/22/localhost-pc-phone/选区_010.png) 
## 修改代码

然后我们打开``index/view/article/preview.html`` 将配置原来的url配置
```
{:ROOT_DOMAIN}/index/article/main?articleId={$articleId}
```

修改为

```
http://192.168.0.101:80/beautifulArtical/thinkphp/public/index/article/main?articleId={$articleId}
```

然后此时我们扫描二维码就可以用微信预览了。

## 解释

当我们输入url的时候电脑会发送给DNS服务器解析出此域名对应的ip地址，然后我们的电脑才会向路由器发送数据包，然后路由器根据目的ip找到相应的电脑。但是如果我们直接输入ip地址，现在我们的手机机会向路由器和本局域网下的所有电脑发送数据包，然后我的电脑和路由器会同时接收到这个数据包，但是路由器会丢弃这个数据包，因为路由器根据转发表发现此ip地址位于本局域网的一台电脑，而我的电脑则会向我的手机发送具体的数据包，这样手机上就可以访问localhost的网页内容了.

## 总结

我本已经学过了计算机网络，这是最简单的道理，但自己却没有想到。用课本知识解决实际问题的能力还是太差。
