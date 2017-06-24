---
title: 计量项目建立路由的流程
date: 2017-06-23 21:32:08
tags: [yoeman,route,angularjs]
category: chenyuanyuan
---
通过这一段时间建立路由，明白了项目的整体文件之间的关系。下面以本次项目为例说一下在angularjs中建立路由的流程。

# 确定菜单名称
将需求分析给出的原型中，每一个菜单名称翻译出来。可以以本次的计量项目需求为例
![](/images\2017\06\23\how-to-establish-route/page1.jpg)
确定好每个菜单名对应的英文名称后，开始进行路由的建立

<!--more-->

# 采用yoeman建立路由
打开`git shell`进入到本项目的`Webapp`文件下，输入下面命令
```shell
yo angular:route supervise/organization/index --uri=supervise/organization
```
这句命令的意思是，建立“监督抽查-授权检定机构监督抽查”这个菜单相对应的`controller`、`view`以及`test`中的文件。
>上面建立路由的方式是用`yoeman`对`angular js`建立路由的方式，但是由于本项目中套用了模板的`js`，所以按照上面的语句建立路由以后，需要手动对项目文件中的`scripts/app.js`、`scripts/config.js`等文件进行修改。

## 修改scripts/app.js文件
将本文件中因`yoeman`建立路由而生成的路由删掉。即删掉下述部分
![](/images\2017\06\23\how-to-establish-route/page2.jpg)

## 修改scripts/config.js文件
在本文件中，添加相应的路由
![](/images\2017\06\23\how-to-establish-route/page3.jpg)

## 修改控制器文件名
此时，为了保证文件名和控制器名称保持一致，还需要修改一下控制器的文件名，即将
![](/images\2017\06\23\how-to-establish-route/page4.jpg)
中的`index.js`改成`indexController.js`，然后在`scripts/index.html`中将
![](/images\2017\06\23\how-to-establish-route/page5.jpg)
也对应的改为`indexController.js`。

这样，我们就完成了整个路由建立的过程。

------
感谢您花费时间阅读这份总结，有不对之处欢迎指正。