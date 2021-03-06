---
title: FreeBSD中，重新启动并在后台运行jar文件
date: 2017-06-19 16:06:11
tags: 
- freebsd
- backgroud
- forever
category: teacherPan
type: "tags"
---
在`freebsd`中，如果我们以`ssh`进行登录，直接启动`jar`文件的话，会发现，当窗口关闭后，运行的程序也被自动终止了。怎么样才能实现当对话关闭后，程序仍将继续运行呢？

# 运行程序
我们使用`mvn package`对应用程序进行打包后，后得到一个可以执行的`.jar`文件，可以在命令行中，使用`java -jar`来运行该`jar`文件。在进行后台运行时，需要在最后面加入`&`，比如:
```shell
# java -jar api-0.0.3-SNAPSHOT.jar --datasource.port=3633 &
```

<!--more-->
# 终止程序
我们知道当运行程序时，可以按`ctrl+c`来终止程序。那么，如何终止正在后台运行的程序呢？

## 找到程序的pid
```shell
# ps -aux
USER     PID  %CPU %MEM     VSZ    RSS TT  STAT STARTED         TIME COMMAND
...
root   90092   0.0  0.1   23572   1200  2- I    Tue10AM      0:00.05 _su (csh)
root   90366   0.0  0.1   23572   2492  3- I    Tue10AM      0:00.02 _su (csh)
root   90367   0.0  1.7  653332  35184  3- I    Tue10AM      0:02.49 node /usr/local/bin/http-server /mengyunzhi/webapp/8005/dist/ -p 8005
root   94194   0.0 15.1 2151736 313104  4- S    Wed05PM      9:21.50 /usr/local/openjdk8/bin/java -jar api-0.0.3-SNAPSHOT.jar --datasource.port=3633

```
我们发现，前面我们在后面执行过的命令的pid为94194
## 终止pid
```sheel
# kill -9 94194
```

# 总结
1. 将程序设置为后台运行，在窗口关闭时，程序仍然将自动运行。
2. 后台运行的程序，需要使用`kill -9`来终止进程。
3. 在使用`kill -9`来终止进行前，需要使用`ps -aux`来查看所想终止程序的进程号。

<hr />
如果是nodejs的应用程序呢？比如http-server.

如果是启动`nodejs`中的程序，比如`http-server`，则需要使用`node`中的`forever`来
比如： 
```bash
# npm install -g forever
# forever start `http-server /mengyunzhi/api/8080.Measurement/dist/ -p 8005` &
```

