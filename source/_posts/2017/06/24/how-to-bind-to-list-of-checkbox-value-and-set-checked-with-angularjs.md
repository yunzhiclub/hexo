---
title: 在angularJs中，如何将list列表数据绑定到一系列的checkbox中。
date: 2017-06-24 22:59:50
tags: [angularJs, checkbox]
category: teacherPan
---
在项目开发中，我们经常会遇到多到多的数据更新操作。比如，在一个用户可以拥有多个角色的系统中，我们需要更新某个用户的角色的操作。
<!--more-->
# 目标
在进行数据更新前，我们需要知道，哪些角色是该用户已经拥有的，哪些角色又是该用户还没有拥有的。
那些已经拥有的角色，我们需要设置为默认选中；而那些还没有的角色，我们需要给出列表，但不能选中。
比如，我们现在编辑的用户，只拥有“器具用户”角色，那么我们期望的初始状态如下：
{% asset_img 1.png 为用户选择角色%}

# 数据前提
想有上图中的效果，我们必须有两项基本的数据支持：
1. 系统共有多个种角色`roles`
2. 当前用户拥有的角色`user.roles`
对于上图而言：
`roles` = [技术机构,器具用户,系统管理员，管理部门，管理员];
`user.roles` = [器具用户];

# 解题思路
* 将`user.roles`进行转换，做为该用户拥有的角色。
* 循环输出`roles`。
* 在每个循环子项中，判断当前`role`是否存在于`user.roles`中。
    * 存在，说明用户拥有此权限，选中。
    * 不存在，说明用户并不拥有此权限，不选中。
* 点击checkbox时，重新设置用户拥有的角色。
* 提交数据时，将用户拥有的角色数组进行逆转换。

# 代码
## 将`user.roles`进行转换，做为该用户拥有的角色。

```js
self.roleIds = [];    // 角色ID数组
      
// 将角色的ID单独取出，新建数组
angular.forEach(self.data.roles, function(value) {
    self.roleIds.push(value.id);
});
```
## 循环输出`roles`

```html
<div ng-repeat="role in roles">
    <label><input type="checkbox">&nbsp;&nbsp;{{role.name}}</label>
</div>
```
## 在每个循环子项中，判断当前`role`是否存在于`user.roles`中。
* 存在，说明用户拥有此权限，选中。
* 不存在，说明用户并不拥有此权限，不选中。

```html
<div ng-repeat="role in roles">
    <label><input type="checkbox" ng-checked="checked(role)">&nbsp;&nbsp;{{role.name}}</label>
</div>
```

```js
// 检测是否选中角色
self.checked = function(role) {
    // self.roleIds
    if (self.roleIds.indexOf(role.id) === -1) {
        return false;
    } else {
        return true;
    }
};
```
## 点击checkbox时，重新设置用户拥有的角色。

```html
<div ng-repeat="role in roles">
    <label><input type="checkbox" ng-checked="checked(role)" ng-click="toggleSelection(role)" >&nbsp;&nbsp;{{role.name}}</label>
</div>
```

```js
      // 用户切换角色时，实时的用户选中的角色信息进行更新
      self.toggleSelection = function(role) {
         var idx = self.roleIds.indexOf(role.id);
          if (idx === -1) {
              self.roleIds.push(role.id);
          } else {
                self.roleIds.splice(idx, 1);
          }
      };
```

## 提交数据时，将用户拥有的角色数组进行逆转换。
```js
self.submit = function() {
            var roles = [];
            // 进行逆转换
            angular.forEach(self.roleIds, function(value){
                roles.push({id:value});
            });
            $scope.data.roles = roles;
            UserServer.update($scope.data.id, $scope.data, function(status){
                if (204 === status) {
                    $location.path('/system/Userfile');
                } else {
                    console.log("todo:提示错误信息");
                }
            });
      };
```

# 总结：
* 想实现基本的功能，需要有两个数据支持。
    * 所有的角色
    * 用户拥有的角色
* 要显示所有的角色列表，所以要对角色进行循环输出
* 每输出一个角色，都需要用方法来决定是否进行选中
* 用户每选中一个角色，都需要更新数据
* 在进行数据提交前，要转换为后台可以接收的数据规范