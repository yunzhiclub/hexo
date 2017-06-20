---
title: 复写Bower.json文件比较好的方法
date: 2017-06-20 20:26:29
category: liuxi
tags: hexo
---
## Override Bower Packages
当我用 bower install 下载一个github上的 bower package时，下载完成之后，它自己带的bower.json文件里面并没有 "main": "angular-locale_zh-cn.js"这种使得我们启动 grunt项目 来帮我们直接执行添加依赖的代码。 当然，我们自己可以在自己本地的bower.json文件中添加 这行语句，这样我们在本地可以 在执行grunt server 的时候，可以在我们设置的index.html文件中自动引入 angular-locale_zh-cn.js这个文件。 但是问题来了，我们下载的bower_components 文件是被在我们项目里面 ignore的，因此并不能同步到我们 所在的github项目中，对此我们得找出解决方案。

我们在启动项目的时候会自动 执行项目目录下的bower.json 文件，我们可以在项目的bower.json文件中直接复写 我们下载的本地的 bower.json文件。示例：

![](/images/Override Bower.json.png)

这样，我们就可以 在启动项目 的时候 自动引入我们下载好的angular-locale_zh-cn.js文件。

当然，这个同样适用于 我们bower所下载的包里面的 bower.json文件中含有"main"的代码的情况，例如我们上图中给出的bootstrap，我们直接在项目的 bower.json文件中 复写就可以了。
