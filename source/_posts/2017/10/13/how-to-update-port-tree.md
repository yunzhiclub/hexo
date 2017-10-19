---
title: FreeBSD如何更新freebsd的port tree
date: 2017-10-13 16:26:23
tags: freebsd
category: teacherPan
---
在使用freebsd系统进行软件安装时，如果该软件发生了安全问题，则freebsd的ports服务器会更新其对应的版本。此时，如果我们的本地服务器未更新到最新的ports，则会提示“该软件发现了安全问题，请更新ports tree后再来安装”的提示。

# 问题描述
比如，我在安装nodejs时，报了以下错误：
```bash
===>  node-8.1.3 has known vulnerabilities:
node-8.1.3 is vulnerable:
node.js -- multiple vulnerabilities
WWW: https://vuxml.FreeBSD.org/freebsd/3eff66c5-66c9-11e7-aa1d-3d2e663cef42.html

1 problem(s) in the installed packages found.
=> Please update your ports tree and try again.
=> Note: Vulnerable ports are marked as such even if there is no update available.
=> If you wish to ignore this vulnerability rebuild with 'make DISABLE_VULNERABILITIES=yes'
```
我们常说小白，小白只所以白，往往是由于我们发现错误后，根本就不看错误给我的提示，然后就胡乱的进行一通错误的排查。而错误提醒却恰恰是解决这个问题关键。

是的，我们往往在开发程序时，也会把一些重要的话输出出来，来提醒使用者他那底错在哪了。告别小白，从看错误提示开始.
<!-- more -->

解决的方法，当然是上述的提示进行操作了。

> Please update your ports tree and try again =》 请更新你的ports tree后再次尝试。

# GOOGLE关键字

有了上面的错误提示，此时，我们google时，方向就更明确了。我们以freebsd update ports tree来进行搜索。很快，将会获得我们想要的答案。比如，我查询时，获取了以下信息：


> Portsnap is the tool we will use to update our ports tree. It is fast, and simple to use. The first time you run portsnap, it needs to download the entire ports tree, which is a download in the tens of megabytes.

> `portsnap fetch`
> `portsnap extract`

>Henceforth, anytime you want to update your ports tree, you will only have to run this command:

> `portsnap fetch update`

碰到英文如果读不懂的话，就慢点读，找自己认识的单词读，先尝试了解这个文章的大概意思。当然，最重要的还是坚持。如果对比英文水平，应该是大一的水平最强，越往后水平越差。如果你不幸在大一大二期间没有阅读过英文的站点，这时候你选择的不应该是放弃，而是比别人多下一番功夫。

我简单的译一下上面的英文：

> 我们可以使用Portsnap这个工具来更新我们的ports tree。Portsnap使用起来即简单又高效。如果你在本服务器上第一次使用Portsnaphe，则需要下载整个ports tree。当然了，下载的东西会很多。以下是首次使用的两个命令。

> `portsnap fetch`
> `portsnap extract`

> 当下次我们再更新ports tree时，只需要使用以下命令就可以了。

> `portsnap fetch update`

# 总结
问题并不复杂。解决问题的难易取决于英文阅读能力的高低。
简单的事情重复做，你就是专家。在平常的学习时，我们离不开使用google查找一些问题。当某个问题你是参考某个英文站点解决的，那么请多花些时间在你刚刚参考的英文页面上，尝试性的翻译大多数的英文单词，来彻底的搞明白这个帮你解决了问题的页面到底说了些什么。

