---
title: 如何启动团队的hexo的博客
date: 2017-10-19 09:44:17
tags: hexo
category: teacherPan
---
本文中，我们将说明，如果在本地配置团队的hexo博客
工具准备： `nodejs`, `git`
<!-- more -->

# clone 项目
`git clone https://github.com/yunzhiclub/hexo.git hexo`

进入项目目录
`cd hexo`

新建自己的分支
`git checkout -b <newBranch>`

> `<abc>` 表示此项为必填项，在实际使用中，将abc替换为自己的内容。

# 安装依赖
`npm install`
`bower install`

# 启动项目
`hexo server`
项目启动后，将在控制台收到
```bash
INFO  Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.
```
我们可以找开`http://localhost:4000/`来查看项目，并在控制台中使用`Ctrl+C`来终止服务。

# 新建文章
`hexo new title`
将新建一篇文章，控制台同时提醒我们新建文章的位置。

最后，编辑文章，保存，将新的文章添加到git中，并提交`pull request`

# 参考文档
![https://hexo.io/zh-cn/docs/](https://hexo.io/zh-cn/docs/)

