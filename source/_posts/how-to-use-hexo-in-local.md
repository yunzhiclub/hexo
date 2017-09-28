---
title: 如何在本地使用hexo发表文章
date: Thu Sep 28 08:57:14 2017
tags: hexo
category: teacherPan
---

# 安装hexo

`hexo`是nodejs下的一个小软件，我们需要使用npm进行安装.

```
$ npm install -g hexo-cli
```

> `hexo`在创建新项目时，会使用`git`命令。如果你是新队员，那么在安装`hexo-cli`前，还需要安装`github`或`git`。

# clone团队项目
项目地址： `https://github.com/yunzhiclub/hexo`
clone后，使用命令行，进入所在文件夹。进行`npm`依赖安装：

<!--more-->

```
$ npm install
```
clone主题
团队blog的主题在原第三方主题的基础上，进行了个性化的设置。设置后的内容上传到了团队的仓库中，当前团队应用的主题为：hext(具体可以查看根目录下的.travis.yml文件)。
我们依然在项目的根目录下，执行以下命令。
```
$ git clone --depth=1 https://github.com/mengyunzhi/hexo-theme-next themes/next
```
操作后，你将在`themes`文件夹上得到一个`next`文件夹，结构如下：
```
├── source
│   ├── _posts
│   ├── categories
│   ├── images
│   └── tags
└── themes
    └── next
```


# 新建分支
在项目master分支的基础上，新建自己的分支。
```
$ git checkout -b yourBranchName(替换为你分支的名字，比如xiaoming)
```

# 启动项目
```
$ hexo server
```

项目启动后，我们便可以在浏览器中打开`http://localhost:4000/`来实时阅览项目了。

# 新建文章
```
$ hexo new "new title"
```
这里标题尽量使用英文。命令成功执行后，将在`source/_posts`下生成新的文章。

比如：
```md
---
title: new title
date: 2017-05-31 16:57:35
tags: 
category: 
---
```
> 这里，我们尽量的采用英文的名称先进行命名，因为它会自动我们生成文件名。在文件名中保证不出现中文，是个好个习惯。

然后：我们将这里的`title`修改为中文。`tags`是标签，如果是多个，可以这样写:`[html,css]`。
`category`类别，这里写上自己的名字。

```md
---
title: 如何在本地使用hexo发表文章
date: 2017-05-31 16:57:35
tags: hexo
category: teacherPan
---
```

最后，就是我们使用markdown语言进行BLOG的写作了。

内容完成后，像我们参与其它的团队项目的开发一样，进行`pull request`。代码审核后，你的文章将自动的出现在[http://www.mengyunzhi.cn](http://www.mengyunzhi.cn)中.

---
更多内容请参考 ：[https://hexo.io/docs/index.html](https://hexo.io/docs/index.html)