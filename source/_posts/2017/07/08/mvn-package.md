---
title: mvn package命令
date: 2017-07-08 17:25:39
categories: gaoliming
tags: [maven]
---

``maven``是托管java项目的一个很好用的工具。但是对于初学java的我来说，也踩到了许多坑。简单说一下我在打包的时候遇到的问题。
<!--more-->

## Find question

当时我执行``mvn package``之后就发现测试文件好多都报错了。

![](/images/package_error.png) 

然后，我们一点儿一点儿着错误就会发现有报错信息

```
A Foreign key refering com.mengyunzhi.measurement.repository.ApplyType from com.mengyunzhi.measurement.repository.WorkflowTypeApplyType has the wrong number of column. should be 3
```

## Analysis question
分析这句话就会发现值WorkflowTypeApplyType这张表里的外健约束from ApplyType有问题。

这个可能是因为我们开发的时候往往是在自己的分支上开发，但是一些简单组件的测试是由开发人员自己做的，我们进行``unit test``的时候已经编译了改动的文件。但是切到``development``分支的时候进行``mvn package``,然后过程如下：
![](/images/package.png) 

虽然我们进行了编译，但是可能会留下曾经编译过的字节码文件(.class)。所以我们可以``rebuild project``, 将所有的编译文件clean然后全部重新编译。

## Solve question
删除数据库，新建数据库。在linux, IDEA开发环境下 **Build -> Rebuild Project**， 然后再执行``mvn package``

## New question

不会报错了之后，当我们进行打包的时候就会发现一个问题。我的电脑总是卡在这句话上

```
HHH000227: Running hbm2ddl schema export
```

这句话表示``hibernate``按照持久化类和映射文件自动生成数据库架构, 把DDL脚本输出到标准输出，同时/或者执行DDL语句。

应为在``mvn  package``中我们可能会有多次这条语句，然后我们怎么解决这个慢的问题呢？

+ 设置 spring.jpa.hibernate.ddl-auto=create
+  随便执行一个单元测试应该也可以,当执行过
```
org.hibernate.tool.hbm2ddl.SchemaExport  : HHH000227: Running hbm2ddl schema export
 2017-07-07 09:18:35.672  INFO 7652 --- [           main] org.hibernate.tool.hbm2ddl.SchemaExport  : HHH000230: Schema export complete 
```
 说明数据表已经创建成功, 即可终止单元测试
+  修改：spring.jpa.hibernate.ddl-auto=none,禁用禁用ddl-auto
+  执行mvn test或者mvn package

然后就会解决这个建立数据库结构慢的问题了。

## 总结

我们总会遇到各种各样的问题，当我们有了一定的知识积累之后，我们就需要对遇到的问题一点点儿分析。不怕慢，就怕我们自己对未知事务的恐惧会阻挡我们前进。当然了，对于问题，我们最好先猜测一个原因再去百度问题的解决方法，因为过多的依赖百度会让我们停止对问题的思考。
## 参考文章
陈鋆:  [Maven 各命令执行流程解析和说明](Maven 各命令执行流程解析和说明) 
[[Nhibernate]SchemaExport工具的使用（一）通过映射文件修改数据表](http://www.it165.net/admin/html/201411/4226.html) 