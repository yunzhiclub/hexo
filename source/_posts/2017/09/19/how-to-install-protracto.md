---
title: 安装protracto记
date: 2017-09-19 15:13:34
tags: ['angularjs', 'protracto']
category: teacherPan
---
`protracto`是`angularjs`下的一个`E2E`集成测试工具。所谓集成测试，就是把各个模块开发完，对接好后，一起跑一下系统，然后看各个模块的关联是否有问题。

我们在项目中，往往通过手工点击的方法来完成集成测试。由于每个人对项目的理解都不一致，加之每次测试的状态及我们所处的环境不同。导致了前后可能测试了100次，但却使用了100次不同的测试步骤。

有了集成测试工具，我们只需要把相关点击和输入的操作放在代码中，然后看着浏览器自动为我们执行操作就可以了。

我们还可以像单元测试一样，写一些断言。当测试跑完，系统会自动告诉我们哪些测试没有通过，期待的值是什么，又实际返回了什么值。

<!--more-->
# 官方文档
我进行安装前，首先，我找到了官方文档：
[http://www.protractortest.org/#/tutorial](http://www.protractortest.org/#/tutorial)

然后就是按照官方文档的步骤进行安装。
官方文档的介绍，大体分为以下几步。
1. 安装protractor
2. 升级webdriver-manager
3. 启动webdriver-manager
4. 写测试文件
5. 写配置文件
6. 运行第一个测试
7. 完善测试

首先，我们先进入`angularjs`的项目根目录，比如`webApp`。

# 安装protractor
安装protractor，我们需要使用命令：
`npm install -g protractor`
这代码着，`protractor`依赖于`nodejs`.

# 升级webdriver-manager
使用命令：
`webdriver-manager update`

实际的安装中，我遇到了以下问题：
* `nodejs`版本较低，导致输入上述命令时，提示语法错误（高版本新的语法，低版本的`nodejs`不支持）
* 提示发生了`access`权限问题
* 在线更新包时，发生了超时连接的错误

解决方法如下：
* 进入`nodejs`官网，下载最新版本的`nodejs`并安装.
* 在使用命令时，使用`sudo webdriver-manager update`（windows权限问题，请`google`解决）
* 由于我使用了`shadowsocks`做为代理，所以在围绕`shadowsocks`作文章。
    * 使用`brew install privoxy`命令，安装了并配置了`privoxy`，将`socks`代理变为`http`代理。
    * 使用`webdriver-manager --help`命令，获取到我可以使用`--proxy`参数来指定代理。
    * 使用`sudo webdriver-manager update --proxy=http://127.0.0.1:8118`来完成了更新操作.

# 启动webdriver-manager
同样，我使用了官方的`webdriver-manager start`来尝试启动这个应用。运气不太好，获取了无法执行的错误。`google`查询后，得到了当前路径的`node_modules`中不存在`webdriver-manager`导致出错的原因。观察`package.json`发现，的确未自动写入关于`protracto`。再经过查询，得到以下文章，并主要进行了参考：

[http://thejackalofjavascript.com/end-to-end-testing-with-protractor/](http://thejackalofjavascript.com/end-to-end-testing-with-protractor/)

略过前面安装`yoman`的部分，分别执行以下语句：
`npm i protractor --save-dev`
`sudo ./node_modules/protractor/bin/webdriver-manager update --proxy=http://127.0.0.1:8118`
`sudo ./node_modules/protractor/bin/webdriver-manager start`
服务成功启动。

> 我们将服务窗口最小化(注意：不是关闭)，然后继续操作。

# 写测试文件
在`test`文件夹中，新建`e2e`文件夹，及`login`文件。

{% asset_img 1.png 新建文件 %}

```js
describe('Protractor Demo App', function() {
  it('should have a title', function() {
    browser.get('');        // 打开根地址（相对于配置文件的baseUrl）
    expect(browser.getTitle()).toEqual('计量器具管理平台'); // 断言标题
  });
});
```

# 写配置文件
在根目录中，我们建立`protractor-e2e.js`，并加入以下配置：
```js
exports.config = {
    // 运行所有的脚本的最长时间
    allScriptsTimeout: 99999,
    // The address of a running selenium server.前面，我们使用start命令启动的服务地址
    seleniumAddress: 'http://localhost:4444/wd/hub',

    // Capabilities to be passed to the webdriver instance. 测试兼容性的浏览器设置
    capabilities: {
        'browserName': 'chrome'
    },

    // 项目地址
    baseUrl: 'http://localhost:9000/',

    framework: 'jasmine',

    // Spec patterns are relative to the current working directly when
    // protractor is called.
    // 测试文件所在位置（使用的是表达式）
    specs: ['test/e2e/*.js'],

    // Options to be passed to Jasmine-node.
    // 其它选项
    jasmineNodeOpts: {
        showColors: true,
        defaultTimeoutInterval: 30000,
        isVerbose: true,
        includeStackTrace: true
    }
};

```
# 运行第一个测试
`./node_modules/protractor/bin/protractor protractor-e2e.js`
此时，浏览器当自动当前我们测试文件中设置的地址，然后获取标题信息，并进行断言。

# 完善测试
继续完善login.js
```js
'use strict';

/* https://github.com/angular/protractor/blob/master/docs/getting-started.md */

describe('my app', function() {
    browser.ignoreSynchronization = true;
    browser.waitForAngular();

    beforeEach(function() {
        browser.get('');    // 每次测试前，都先尝试打开根目录
    });

    it('自动跳转至用户登录界面', function() {
        expect(browser.getTitle()).toEqual('计量器具管理平台'); // 获取并断言标题
        // 获取当前的url信息，并断言必然跳转至/login
        browser.driver.getCurrentUrl().then(function(url) { 
            var absUrl = url.split('#!');
            expect(absUrl[1]).toEqual('/login');
        });
    });

    it('正确的用户名密码进行登录', function() {
        // 获取username password和登录按钮
        var username = element(by.model('user.username'));
        var password = element(by.model('user.password'));
        var login = element(by.partialButtonText('登'));

        // 自动输入用户名和密码
        username.sendKeys('admin');
        password.sendKeys('admin');

        // 前击登录
        login.click();

        // 延迟3秒等待系统完成登录
        browser.driver.sleep(3000);
        browser.waitForAngular();

        // 获取当前地址为首页的仪表台地址
        browser.driver.getCurrentUrl().then(function(url) {
            var absUrl = url.split('#!');
            expect(absUrl[1]).toEqual('/main/dashboard');
            console.log(url);
        });
    });
});

```

重新启动测试，效果如下图：

{% asset_img 1.gif 新建文件 %}

# 总结
安装环境是一项具有挑战性的工作。它能够考验自己对工具掌握及理解的情况。如果我们在安装环境中碰到了问题，要尝试着去翻译各种提示的英文文字，然后借助`google`去找到问题的原因及解决方案。

注：安装顺利.




