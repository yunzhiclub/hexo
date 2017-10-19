---
title: review(审阅)pull request的方法
date: 2017-10-19 09:58:11
tags: github
cagegory: teacherPan
---
在团队开发的过程中，我们需要将自己开发的代码使用`pull request`的方式提交至`github`中。本文中，我们给出当我们看到别人提交的代码时，如何快速的来审阅这次代码提交。

<!-- more -->
首先，假设我们有以下`pull request`.
{% asset_img 1.jpg pullRequest示例 %}
# 拉取项目
`git pull`

# 找到提交分支
如下图，我们查看到，当前的`pull request`是由`reviewPullRequest`提交上来的。
{% asset_img 2.jpg pullRequest示例 %}

# 切换到提交分支
<hr>
如果我们正在进行同一项目的开发，那么我们可以先将自己的代码进行`commit`。
比如，我们使用以下命令
```bash
git add ./
git commit -a -m "temp"
```
<hr>
切换至提交`pull request`的分支`reviewPullRequest`。

`git checkout <reviewPullRequest>`

# 查看效果或是代码
此时，我们便可以启用本地环境来查看效果了。

如果我们查看是`readme.md`文档，那么还可以直接登录`github`官网，然后在官网上直接切换分支后在线阅览`readme.md`的效果。

{% asset_img 3.jpg pullRequest示例 %}


