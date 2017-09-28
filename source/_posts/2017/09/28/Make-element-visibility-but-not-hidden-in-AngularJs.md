---
title: 在angularjs中，设置元素不可见替代隐藏元素的方法
date: 2017-09-28 13:33:34
tags: AngularJS
category: teacherPan
---
在AngularJS中，我们都知道使用`ng-hide`来隐藏元素。但有时候，我们需要的却是元素不可见。

此时，我们需要使用`ng-style="{'visibility': express ? 'hidden' : 'visible'}"`来代替`ng-hide="express"`。其中，`express`为变量或是表达式。

元素不可见与元素隐藏有什么关系呢？

在`CSS`中:
隐藏元素 = `display: none`
元素不可见 = `visibility: hidden`

下面，我们通过下面的小例子，来看看具体他们的区别，以及元素不可见的应用场景。
<!-- more -->
# 实例
在计量的项目中，我们需要对精确度进行排序，假设现在效果如下：

{% asset_img 1.gif 排序示例图 %}

点击上、下图标时，可以改变精确度的顺序。

# 需求
我们的实际需求是这样的：

{% asset_img 1.png 实际想要的效果 %}

即，使用`visibility: hidden`，不显示（同时，仍然占有原来的位置）第一条的“向上箭头”，不显示（同时，仍然占有原来的位置）最后一条的“向下箭头”。
代码段为：
```html
ng-style="{'visibility': $last ? 'hidden' : 'visible'}"
```
其中，`$last`为变量或是表达式。

# ng-hide
我们将第一条的`visibility: hidden`，更改为`ng-hide`
修改前：
```html                                
<a href="javascript:void(0);" ng-style="{'visibility': $first ? 'hidden' : 'visible'}" ng-click="upAccuracy(data)">&nbsp;<i class="fa fa-caret-square-o-up text-success"></i>&nbsp;</a>&nbsp;

```
修改后：
```html
                                <a href="javascript:void(0);" ng-hide="$first" ng-click="upAccuracy(data)">&nbsp;<i class="fa fa-caret-square-o-up text-success"></i>&nbsp;</a>&nbsp;
```
修改后效果：

{% asset_img 2.png bug效果 %}

我们看到，通过`ng-hide`后，后面的元素视为该元素不存在，所以占用了隐藏元素原来的位置。而使用`ng-style="{'visibility': express ? 'hidden' : 'visible'}"`后，虽然其隐藏掉了，但后面的元素依然视该元素存在。近而保持了格式的统一。

参考：[https://stackoverflow.com/questions/26927585/visibility-hidden-in-angularjs](https://stackoverflow.com/questions/26927585/visibility-hidden-in-angularjs)

google关键字：`ng-hide visibility hidden`



