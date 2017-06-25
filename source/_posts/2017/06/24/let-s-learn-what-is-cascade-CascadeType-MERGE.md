---
title: let's learn what is 'cascade = CascadeType.MERGE'
date: 2017-06-24 23:11:21
tags:
---
通过用户 -- 角色，来演示`cascade = CascadeType.MERGE`的作用.
1. 演示添加该属性时，进行用户编辑时，竟然一同编辑了角色的name.
2. 去除`cascade = CascadeType.MERGE`后再次编辑。正常显示。