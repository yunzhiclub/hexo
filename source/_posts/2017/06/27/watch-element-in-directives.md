---
title: 监听指令的外部数据源变化
date: 2017-06-27 13:57:11
tags:['angularjs', 'watch', 'directive', 'element']
category: teacherPan
---
先给一个效果图。其中区域是一个指令，而用户是另一个指令。我们希望当区域为区县级别时，显示用户选择列表。非区域级别时，隐藏用户选择框。
{% asset_img 1.gif 效果%}

# 分析问题
我们希望在用户指令中，监听区域指令发生的变量。大体思想是这样的。
* 我们将区域做为用户指令的一个属性进行绑定。
* 在指令选择区域后，当前选择**区域**由指令回传给V层。
* 用户指令监听到这个变化，并做出响应

# 解决问题
## 方法一
控制器：
```js
angular.module('webappApp')
    .controller('MandatoryPassrateIndexCtrl', ['$scope', function($scope) {

        $scope.currentDistrict = {};       // 当前区域
        $scope.user = {};           // 当前用户
        $scope.appliance = {};      //当前器具
     
    }]);
```

V层
```html
<div class="col-md-4">
    <div class="row form-group">
        <label class="col-md-4 control-label">区域:</label>
        <div class="col-md-8">
            <yunzhi-district ng-model="currentDistrict"></yunzhi-district>
        </div>
    </div>
</div>
<div class="col-md-4">
    <div class="form-group">
        <label class="col-md-4 control-label">用户:</label>
        <div class="col-md-8">
            <yunzhi-measurement-user ng-model="user" data-district="currentDistrict"></yunzhi-measurement-user>
        </div>
    </div>
</div>
```

用户指令：
```js
angular.module('webappApp')
    .directive('yunzhiMeasurementUser', ['measurementUser', function(measurementUser) {
        return {
            templateUrl: 'views/directive/yunzhiMeasurementUser.html',
            controller: function($scope) {
                console.log($scope);
            },
            link: function postLink(scope, element, attrs) {
                console.log(attrs);
                // 监听传入的data-district是否发生了变化，如果发生了变化，则重新获取器具用户列表
                scope.$watch(attrs.district, function(newValue) {
                //something
                }
```

控制台信息：
{% asset_img 1.png 效果%}

观察控制台，我们发现：我们想监控是`ChildScope`中的`currentDistrict`，该项信息存在于`Attributes`的`disctrict`。所以，我们使用`scope.$watch(attrs.district,` 进行监听。


## 方法二
我们引用局部`scope`变量，并进行双向数据绑定。
```js
angular.module('webappApp')
    .directive('yunzhiMeasurementUser', ['measurementUser', function(measurementUser) {
        return {
            templateUrl: 'views/directive/yunzhiMeasurementUser.html',
            scope: {
                ngModel: '=',            // 双向绑定ngModel
                district: '='            // 双向绑定data-district
            },
            controller: function($scope) {
                console.log($scope);
            },
            link: function postLink(scope, element, attrs) {
                console.log(attrs);
                // 监听传入的data-district是否发生了变化，如果发生了变化，则重新获取器具用户列表
                scope.$watch('district', function(newValue) {
                //something
                }
```

控制台：
{% asset_img 2.png 效果%}

此时，`scope`中的字段信息，为重命名后的字段，所以直接使用字符串`district`, 进行监听。

# 总结：
无论哪种方式，最终我们要监听的数据位于`$scope`上。要监听前，我们需要对`$scope`进行打印，进而找到要监听的对象名称。最终将该名称以字符串的方式传入到`scope.$watch()`中以完成监听过程。

> 调试，无论在哪种语言中，都具有最高的地位！

