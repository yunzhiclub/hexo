---
title: how to build SpringMVC application
date: 2017-06-20 17:55:28
tags: SpringMVC
category: teacherPan
---
我们使用SpringMVC进行项目的开发后，如何生成可以运行的应用程序，又是该用什么方法启动应用程序为我们服务呢？本篇中，我们将简单进行讲解。

# 官方文档
还是我们一直说的：我们想到的，大牛们早就做到了！
Spring的官方guide中的[Building Java Projects with Maven](https://spring.io/guides/gs/maven/)，非常清晰的为我们讲述了如果使用maven构建Java Projects。

鉴于本文在基本于mvn已经成功安装，并且项目已经成功在idea下进行开发的前提下，我们在此，只给出要点。
<!-- more -->
# 清空数据表
为了给项目一个干净的初始化环境，也为了加快项目重构时的创建速度，我们首使用navicat删除当前数据库的所有数据表；或是，我们直接删除数据库，然后新建一个同名数据库。

# 修改配置文件
```
# 在项目初始化时，重新创建数据表
spring.jpa.hibernate.ddl-auto=${jpa.ddl-auto:create}
```
将数据表设置为create方式，即在项目启动时，重新创建数据表，并且初始化系统数据.

# build project
使用git bash进行项目文件夹（即存在pom.xml的文件夹）。
执行命令：
```bash
# mvn package
```
系统将首先进行`mvn test`，测试无误后，将自动在`./target`文件夹中，生成了一个`jar`文件。该`jar`文件即为java应用。

> 如果你想跳过`mvn test`直接生成`.jar`文件，可以使用`mvn package -DskipTests`命令来替换`mvn package`

# 启动应用
我们进入jar文件所在路径，并使用java命令启动`jar`文件
```bash
# java -jar xxxxx.xxx.jar
```
此时，应用程序就被我们启动了。

# 布署到服务器
有了jar文件，布署到服务器就简单了。我们可以使用任何方式来将`jar`文件复制到服务器中。U盘，文件共享，邮件什么都可以。

## 准备数据库
在服务器上启动mysql（可以是xampp这种集成环境，也可以是单独安装的），然后使用navicat创建一个项目使用的数据库（名字要相同）。如果已经有数据库了，可以先删除，然后再创建。

## 执行jar文件
我们在服务器上存放jar文件的路径上启动`git bash`。然后和我们在本机执行过程一样。
```bash
# java -jar xxxxx.xxx.jar
```

程序就这样被启动了！

那么如何重新布署呢？
在执行上述操作前，我们只需要先关掉上次正在执行的`jar`程序就可以了。

# 更改运行参数
有时候，我们服务器的运行环境和本地可能会有些出入。比如服务器的数据库的端口不是3306。而是3633。这时候该怎么办呢？
我们通过观察程序的配置文件不能发现，有很多`${xxx:yyy}`的配置信息，这就是为我们自定义配置准备的。其中`xxx`是配置项，`yyy`是默认值。

```
# 在项目初始化时，重新创建数据表
spring.jpa.hibernate.ddl-auto=${jpa.ddl-auto:create}
# 指定连接的类型为mysql 连接的地址为：localhost 端口为3306 ，数据为springmvc
spring.datasource.url=jdbc:mysql://127.0.0.1:${datasource.port:3306}/measurement?useUnicode=true&characterEncoding=utf-8
# 用户名为root
spring.datasource.username=${datasource.username:root}
# 密码为空
spring.datasource.password=${datasource.password:}
server.port=${port:8080}
```
比如，我们想将数据库的端口号改为3633，则应该使用:
```bash
# java -jar xxx.xxx.jar --datasource.port=3633
```

再比如，我们想将数据库的端口号改为3633的同时，还想将程序的端口修改为8081
则使用：
```bash
# java -jar xxx.xxx.jar --datasource.port=3633 --port=8081
```


# 总结：
1. 使用mvn package将生成一个jar程序。
2. 使用java -jar xxx.jar来运行一个程序。
3. 为了避免数据表不一致带来的问题，提前清空数据表。
4. 程序运行以前，需要确保mysql服务已启动。
5. 程序运行 = mysql服务运行 + jar应用程序运行。
6. 可以在配置文件中加入`${}`来在程序启动时自定义配置信息。










