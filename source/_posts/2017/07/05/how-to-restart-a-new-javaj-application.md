---
title: 如何重启服务器(freebsd系统)上的JAVA应用
date: 2017-07-05 15:55:15
tags: [java application, freebsd]
category: teacherPan
---
本文将阐述如何在`freebsd`系统中，重启`java`应用程序.
<!--more-->
# 获取jar程序
获取jar程序的方法有很多，比如我们可以在本地进行`mvn package`，或是可以直接下载`gh-pages`分支上的`jar`文件。

假设我们拥有以下`jar`文件：
```shell
api-0.0.5-SNAPSHOT.jar

```

# 登录服务器

```bash
$ ssh user@www.mengyunzhi.cn -p 1234 
```
> windows客户端，`ssh`命令端口参考可能不同，请使用`$ ssh -help`来查看自定义端口的参数。

其中`user`为用户名，`www.mengyunzhi.cn`为域名（也可以是IP地址）, `1234`为端口号。回车后，提示我们输入密码。成功后，将显示以下欢迎信息：
```bash
Show the version of FreeBSD installed:  freebsd-version ; uname -a
Please include that output and any error messages when posting questions.
Introduction to manual pages:  man man
FreeBSD directory layout:      man hier

Edit /etc/motd to change this login announcement.
$ 
```

# 停止原应用
```bash
$ ps | grep java
39297  0  R+   0:00.00 grep java
39049  2- R    0:58.06 /usr/local/openjdk8/bin/java -jar api-0.0.5-SNAPSHOT.jar --datasource.port=5678

```

我们得到`pid`号如`39049`，继而使用`kill -9 pid`结束原进程
```bash
$ kill -9 39049
```

操作后，可以继续使用`ps | grep java`来查看进程是否成功结束。

# 上传新应用
利用`sftp`命令进行上传。我们在本机新打开一个命令行或`bash`窗口，然后输入：


```bash
$ cd 本地存放api-0.0.5-SNAPSHOT.jar文件的目录
$ sftp -P 1234 user@www.mengyunzhi.cn
$ 输入密码
Connected to www.mengyunzhi.cn.
sftp> 

```

> windows客户端请先执行`sftp -help`查看如何指定端口及用户名

## 执行上传命令
比如我们服务器上的应用程序存放位置为：`/mengyunzhi/api/8080.Measurement`

首先进行服务器目标文件夹，然后执行`put`命令上传

```bash
sftp> cd /mengyunzhi/api/8080.Measurement
sftp> put api-0.0.5-SNAPSHOT.jar 
Uploading api-0.0.5-SNAPSHOT.jar to /mengyunzhi/api/8080.Measurement/api-0.0.5-SNAPSHOT.jar
api-0.0.5-SNAPSHOT.jar                                                                                                             100%   40MB   2.9MB/s   00:14    
sftp> 
```
传输完成，关闭当前窗口。

# 启动新应用
我们切回到 停止原应用 的窗口，先进入程序所在目录，然后启动新程序。
```bash
$ cd /mengyunzhi/api/8080.Measurement
$ java -jar api-0.0.5-SNAPSHOT.jar --datasource.port=6789 &
```

> `--datasource.port=6789`指定数据库的端口。`&`指定为后台运行，很重要！

服务启动后，控制台将打印：
```
2017-07-05 16:15:01.802  INFO 39339 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8080 (http)
2017-07-05 16:15:01.818  INFO 39339 --- [           main] c.mengyunzhi.measurement.ApiApplication  : Started ApiApplication in 50.57 seconds (JVM running for 51.544)
```







