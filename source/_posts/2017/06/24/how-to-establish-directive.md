---
title: 建立指令的步骤
date: 2017-06-24 11:40:57
tags: [yoeman,directive,route,angularjs]
category: chenyuanyuan
---
以本项目为例，写一下建立指令的步骤。
# 新建指令
在项目的`Webapp`目录下打开`git shell`，用`yoeman`建立新的指令以及指令对应的`test`文件。以新建岗位指令（post）为例。
```shell
yo angular:directive yunzhipost
```
<!--more-->
# 完成指令的内容以及样式
## 完成yunzhipost.js
* 首先，在`yunzhipost.js`中先加入独立`scope`，将指令中的`post`属性双向绑定到`scope.post`上
```javascript
		scope: {
            // 将指令中的post属性，双向绑定到scope.post
            ngModel: '='
        },
```

* 然后指出指令的类型，
```javascript
restrict: 'EA',
```
> 其中，
A（默认）为属性，在html文件中以`<div hello></div>`形式使用；
E为元素，在html文件中以`<hello></hello>`形式使用；
M为注释，在html文件中以`<!-- directive:hello -->`形式使用；
C为`css`样式类，在html文件中以`<div class="hello"></div>`形式使用。

* 然后指明该指令样式所在的模板文件地址，
```javascript
templateUrl: 'views/directive/yunzhiPost.html',
```

* 在`controller`里面，写出该指令实现的方法，包括：初始化、获取后台数据等。
```javascript
controller: function($scope) {
    $scope.posts = []; // 初始化所有岗位
    $scope.post = {}; // 初始化岗位
    $scope.post.selected = $scope.ngModel; // 传值。

    // 获取用户可见的岗位列表
    postService.getCurrentUserPostArray(function(data) {
        $scope.posts = data;
        // 如果大小不为0，而且用户并没有传入ngModel实体，则将第一个岗位给当前岗位
        if (data.length > 0 && angular.equals($scope.ngModel, {})) {
            $scope.post.selected = data[0];
        }
    });
},
```
*在`link`里面，监视内容是否变化，如果发生变化则利用数据的双向绑定，将值传回V层。
```javascript
link: function postLink(scope) {
    // 监视岗位是否发生变化。如果发生变化，则重置ngModel的值。此时，利用双向数据绑定。将值传回V层
    scope.$watch('post', function(newValue) {
        scope.ngModel = newValue.selected;
    }, true);
},
```

## 完成yunzhipost.js对应的V层样式
根据2.1中提到的模板文件地址，在V层的directive文件夹下，新建yunzhiPost.html文件，用于编写该指令对应的V层样式。
```html
<!--岗位-->
<ui-select ng-model="post.selected" theme="bootstrap" ng-disabled="disabled">
    <ui-select-match placeholder="请选择">{{$select.selected.name}}</ui-select-match>
    <ui-select-choices repeat="post in posts | propsFilter: {name: $select.search, pingyin: $select.search}">
        <div ng-bind-html="post.name | highlight: $select.search"></div>
    </ui-select-choices>
</ui-select>
```

# 完成指令的数据读取
在完成以上步骤后，只需要再对后台数据进行读取，这个指令就写完了！
同样的，我们首先用yoeman新建service文件，在service文件中实现对后台数据的获取。
```shell
yo angular:service PostService
```
执行完这条命令以后会自动生成相应的js文件。
以PostService文件为例，我们在service里面定义一个方法，用来获取在data文件夹中写入的模拟json数据。
```javascript
var getCurrentUserPostArray;
getCurrentUserPostArray = function(callback) {
    $http.get('data/post/getCurrentUserPostArray.json').then(function(response) {
        callback(response.data);
    });
};
return {
    getCurrentUserPostArray: getCurrentUserPostArray
};
```
> 与后台数据对接的时候只需要把json数据删掉，将数据获取方式改为后台数据获取即可。

# 将指令写在V层
接下来，我们只要在V层需要的位置写入该指令就可以直接使用了。
```html
<yunzhi-post ng-model="post"></yunzhi-post>
```

------
感谢您花费时间阅读这份总结，有不对之处欢迎指正。