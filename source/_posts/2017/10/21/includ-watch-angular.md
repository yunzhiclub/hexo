---
title: angular中使用ng-include后controller中$watch失去作用
date: 2017-10-21 18:53:51
tags: [angular]
category: gaoliming
---

有的时候为了实现代码复用，我们会使用``ng-include``复用部分代码。

<!--more-->

## 问题简化

``about.js``

```
angular.module('angularTestApp')
  .controller('AboutCtrl', function ($scope) {
      $scope.num = 0;
      $scope.$watch("num", function () {
          console.log($scope.num);
      });
  });
```
``about.html	``
```
<p>This is the about view.</p>
<div ng-include="'views/include.html'"></div>
{{num}}
```

``include.html``

```
<div>
  <input type="text" ng-model="num">
  <p>{{ num }}
</div>
```

如果我们运行这段代码就会发现一个问题，当我们改变``num``的值的时候控制台并没有打印``num``的值。

## 原因

这是因为我们使用``ng-include``的时候会会创建一个``child scope``。然后两个``scope``的作用域是相互隔离的，所以``include.html``中的``ng-model``绑定的是自己的``scope``, 所以``about.js``中的``num``不会发生改变。

## 解决办法

将``include.html``做如下修改

```
<div>
  <input type="text" ng-model="$parent.num">
  <p>{{ num }}
</div>
```

## 参考文献
[ng-model not working inside ng-include](https://stackoverflow.com/questions/32775461/ng-model-not-working-inside-ng-include) 