---
title: php + xdebug + phpstrom断点调试
date: 2017-09-18 15:57:43
tags: [php]
categories: gaoliming
---
我们说``sublime``和``xdebug``调试php程序的时候数组变量有的时候不能显示完全，所以就给我们调试程序带来了不方便，如果使用``var_dump``又太浪费时间（可能是因为本人技术水平不佳）。

<!--more-->

**环境**: Ubuntu16.04, lampp, phpStorm

## download xdebug

我们可以新建一个php文件，然后写上:
```
phpinfo();
```
来查看自己安装php的具体信息，以方便我们找到合适自己的``xdebug``
![](/2017/09/18/php-xdebug-phpstorm/phpinfo.png) 

然后我们打开网址[https://xdebug.org/wizard.php](https://xdebug.org/wizard.php) ``Ctrl + A``全选将自己的php信息输入到空白页面。
点击**Analyse my phpinfo() output**

然后``xdebug``官网就会推荐合适xdebug版本而且给出详尽的安装步骤。


## install xdebug

下面的步骤因为个人安装路径的不同而不同，不用担心，因为``xdebug``官网分析的已经非常到位了，只要能简单读懂英语相信都可以安装成功。

### 解压文件

```
sudo tar -xvzf xdebug-2.5.4.tgz
cd xdebug-2.5.4/
phpize
```

然后我们会发现提示以下错误:

```
程序“phpize”尚未安装。 您可以使用以下命令安装：
sudo apt install php7.0-dev
```
### install php7.0-dev
我们安装提示继续进行:

```
sudo apt install php7.0-dev
sudo phpize
sudo ./configure
sudo make
sudo cp modules/xdebug.so /opt/lampp/lib/php/extensions/no-debug-non-zts-20151012
```

### 编辑php.ini

```
sudo gedit /opt/lampp/etc/php.ini
```

最后一行添加:

```
zend_extension = /opt/lampp/lib/php/extensions/no-debug-non-zts-20151012/xdebug.so
//调试信息代码使用
xdebug.remote_host = 127.0.0.1
xdebug.remote_enable = 1
xdebug.remote_port = 9000
xdebug.remote_handler = dbgp
xdebug.remote_mode = req
```

### 重启服务

```
sudo /opt/lampp/lampp restart
```

然后我们重复第一步查看自己的php信息发现``xdebug``安装成功
![](/2017/09/18/php-xdebug-phpstorm/xdebug.png) 

## install Xdebug extension helper

### google chrome

插件安装地址
[https://chrome.google.com/webstore/detail/xdebug-helper/eadndfjplgieldjbigjakmdgkmoaaaoc?hl=en ](https://chrome.google.com/webstore/detail/xdebug-helper/eadndfjplgieldjbigjakmdgkmoaaaoc?hl=en ) 

因为``google``被墙了，如果想要安装所以请自行在``github`` or ``老D``上下载``hosts``。

### IDEKey

在``Xdebug extension helper``的``选项`` 中 ``IDE key``   中选择``phpStorm``。
![](/2017/09/18/php-xdebug-phpstorm/xdebug_helper.png) 

开启``Xdebug extension helper``调试模式
![](/2017/09/18/php-xdebug-phpstorm/xdebug_start.gif) 
## 配置PhpStorm

然后我们打开phpStorm中``Run`` , ``Start Listening for PHP Debug Connections.``
然后我们就可以调试了:
![](/2017/09/18/php-xdebug-phpstorm/break_point.png) 

## 参考文献
[[PHP+xdebug] 在Ubuntu 14.04下的PhpStorm中配置xdebug调试环境](http://blog.csdn.net/dm_vincent/article/details/44678347) 
[Zero-configuration Web Application Debugging with Xdebug and PhpStorm](https://confluence.jetbrains.com/display/PhpStorm/Zero-configuration+Web+Application+Debugging+with+Xdebug+and+PhpStorm) 