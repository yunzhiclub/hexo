---
title: 如何使用U盘启动的方式更新macos
date: 2017-10-20 23:58:32
tags: macos
category: panjie
---
苹果系统出现问题，我们往往可以很快的将系统恢复为出厂设置。
官方教程： [https://support.apple.com/zh-cn/HT204904](https://support.apple.com/zh-cn/HT204904)

那么当我们想对原有系统进行升级呢？
此时，我们就需要下载macos的新系统，然后采用U盘的形式安装了。

<!-- more -->
参考官方英文教程： [https://support.apple.com/en-us/HT201372](https://support.apple.com/en-us/HT201372)

简单描述下步骤：
# 格式化U盘
需要注意两点：
1. 格式应该选择：Mac OS 扩展（日志式）
2. 名称应该起个简短的，不要有空格。比如起名为：OSX

{% asset_img 2.png 说明 %}

# 将系统写入U盘
1. 双击下载的dmg文件，并进行安装。
2. 安装成功后，将在 所有程序 中看到一个名为： 安装 mac os xx

{% asset_img 1.png 说明 %}

3. 打开 安装 mac os xx，但不要点继续，就打开放着就可以。
4. 确认自己当前系统的版本
点小苹果，关于本机。

5. 使用`createinstallmedia`命令进行U盘写入
不同的系统，`createinstallmedia` 不同，命令也不完全相同，这个需要参考官方文档。

我的方法是，输入`/Applications/Install`后，按tab键自动补全，然后与官方文档相对应，就能清楚自己的版本号了。

6. 替换命令中的U盘名称
比如我们的起名为OSX, 则将命令中的 /Volumes/MyVolume 换为 /Volumes/OSX

当然了，你也可以把U盘起名为：MyVolume，这样在输入命令时就不需要变更了。

示例如下：
```bash
bogon:Applications apple$ sudo Install\ macOS\ High\ Sierra.app/Contents/Resources/createinstallmedia --volume /Volumes/OSX/
Password:
Ready to start.
To continue we need to erase the volume at /Volumes/OSX/.
If you wish to continue type (Y) then press return: Y
Erasing Disk: 0%... 10%... 20%... 30%...100%...
Copying installer files to disk...
Copy complete.
Making disk bootable...
Copying boot files...
Copy complete.
Done.
```

然后，我们就可以按[https://support.apple.com/zh-cn/HT204904](https://support.apple.com/zh-cn/HT204904)的说明来重新安装操作系统了。

<hr>
以下内容是关于组建`funsion driver`的
`funsion driver` = 一块机器 + 一块SSD。组建容量高速度快的硬盘驱动器。
<hr>
不更新系统，只是想组建funsion driver，请直接参考官网地址，如下
https://support.apple.com/zh-cn/HT207584
