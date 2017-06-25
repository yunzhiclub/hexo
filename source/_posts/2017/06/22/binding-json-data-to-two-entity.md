---
title: Spring MVC 绑定json数据到多个实体上
date: 2017-06-22 21:17:16
tags: [spring mvc,data binding]
---
在spring mvc的数据绑定定，我们可以轻松的将json对象绑定到一个实体上。但如何一次性的绑定到多个实体上的呢？
<!-- more -->
# 绑定单个实体
绑定一个实体的时候，我们往往是这么做的：
比如，我们在新增用户时，需要绑定传入的用户信息:
```java
 @PostMapping("/save")
    @ApiOperation(value = "save 保存用户",nickname = "User_save",notes = "保存用户")
    public User save(@ApiParam(value = "用户") @RequestBody User user){
        userService.save(user);
        return user;
    }
```

对应的，我们可以这样发送数据，进而完成数据绑定：
```json
{"username":"yunzhiclub", "password":"123456"}
```

# 问题描述
我们刚刚描述了绑定单个实体的方法，但如果我们需要一次发送多个实体到后台，又该怎么办呢？
比如，我们需要把部门和用户信息同时传给控制器做外理。
例如，前台我们将发送这样的数据：
```json
{
    "user":{"username":"zhangsan"},
    "deparment":{"name":"部门1"}
}
```
其中`user`是一个实体的信息，而`department`是另一个实体的信息。
 
# 解决方法
## 新建内部静态类
比如我们现在的控制器为`UserController`,则在`UserController`中建立如下**静态内部类**：
> 注意：必须声明为**静态内部类**，否则将得到一个未能成功绑定的400异常

```java
 @ApiModel(value = "部门+用户")
    public static class DepartmentUser {
        @ApiModelProperty("部门")
        private Department department;
        @ApiModelProperty("用户")
        private User user;
        public DepartmentUser() {
        }
        public Department getDepartment() {
            return department;
        }
        public void setDepartment(Department department) {
            this.department = department;
        }
        public User getUser() {
            return user;
        }
        public void setUser(User user) {
            this.user = user;
        }
    }
```
有了这个内部静态类，我们就可以将数据成功的绑定到上面两个内部实体上了。

```java
    @PostMapping("/register")
    @ApiOperation(value = "register 注册后户", notes = "用户注册时，同时注册部门及用户信息", nickname = "User_register")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void register(@ApiParam(value = "用户注册信息：部门信息+用户信息") @RequestBody DepartmentUser departmentUser) {
        return;
    }
```
此时，`departmentUser`就是我们绑定的信息。我们可以使用其中的`getUser()`和`getDepartment()`来获取绑定的值。我们还可以在`return`上打一个断点，启动调试模式来查看绑定的`departmentUser`的值.
{% asset_img 1.png 调试信息 %}

# 总结
1. json数据可以绑定到某**一个**实体类上。
2. 实体类的本质是类
3. json数据可以绑定到任意类上。
4. json数据绑定到类的前提是：有相应字段的set方法。
5. 要想绑定多个实体，需要将多个实体组合为一个实体。
6. 发送数据时，将不同的实体数据做为json对象的不同属性发送。

# 代码片断

> [问题相关项目代码](https://github.com/yunzhiclub/MeasurementItemFrontEnd/blob/014fc96f75373e07a7d61d150b1e9beb7fb6b64e/api/src/main/java/com/mengyunzhi/measurement/controller/UserController.java), [单元测试代码](https://github.com/yunzhiclub/MeasurementItemFrontEnd/blob/78a6e661bfd49c97d9d5fa9181bd493aa256e531/api/src/test/java/com/mengyunzhi/measurement/controller/UserControllerTest.java)

```java
package com.mengyunzhi.measurement.controller;

import com.mengyunzhi.measurement.Service.UserService;
import com.mengyunzhi.measurement.repository.Department;
import com.mengyunzhi.measurement.repository.User;
import com.mengyunzhi.measurement.repository.WebAppMenu;
import io.swagger.annotations.*;
import org.apache.log4j.Logger;
import org.hibernate.ObjectNotFoundException;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.web.authentication.logout.SecurityContextLogoutHandler;
import org.springframework.web.bind.annotation.*;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.security.Principal;
import java.util.List;

/**
 * Created by panjie on 17/5/17.
 *
 */
@Api(tags = "User 用户", description = "用户管理")
@RequestMapping("/User")
@RestController
public class UserController {
    static Logger log = Logger.getLogger(UserController.class.getName());

    @Autowired
    private UserService userService;

    @GetMapping("/login")
    @ApiOperation(value = "login 登录", nickname = "User_login",
            notes="用户登录成功后，将会得到一个header 信息。客户端存储该值后，下次发送时做为header中的XXX进行发送")
    public void login(Principal user) {
        log.info("-----------------登录成功----------------");
        log.info("登录用户:" + user.getName());
        // 此函数触发，则说明登录成功
        return;
    }

    @GetMapping("/logout")
    @ApiOperation(value = "logout 注销", nickname = "User_logout",
            notes = "参考资料：http://docs.spring.io/spring-security/site/docs/current/apidocs/org/springframework/security/web/authentication/logout/SecurityContextLogoutHandler.html")
    public void logout(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) {
        log.info("-----------------注销----------------");
        // 获取当前认证信息
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();

        // 如果存在认证信息则调用注销操作
        if (null != authentication) {
            new SecurityContextLogoutHandler().logout(httpServletRequest, httpServletResponse, authentication);
        }

        return ;
    }

    @GetMapping("/getAll")
    @ApiOperation(value = "getAll 获取所有用户",nickname = "User_getAll",notes = "获取所有用户")
    public List<User> getAll(){
        return userService.getAll();
    }

    @PostMapping("/save")
    @ApiOperation(value = "save 保存用户",nickname = "User_save",notes = "保存用户")
    public User save(@ApiParam(value = "用户") @RequestBody User user){
        userService.save(user);
        return user;
    }

    @GetMapping("/get/{id}")
    @ApiOperation(value = "get 获取一个用户",nickname = "User_get",notes = "获取一个用户")
    public User get(@ApiParam(value = "实体id") @PathVariable Long id){
        return userService.get(id);
    }

    @ApiOperation(value = "update (修改)", notes = "修改某条数据", nickname = "User_update")
    @ApiResponse(code = 204, message = "更新成功")
    @PutMapping(value = "/update/{id}")
    public void update(@ApiParam(value = "实体ID") @PathVariable Long id, @ApiParam(value = "用户实体") @RequestBody User user, HttpServletResponse response) {
        log.info("---- 更新实体 -----");
        log.info("在数据更新中，可以直接调用M层的save方法.");
        try {
            userService.update(id, user);
            response.setStatus(204);
        } catch (ObjectNotFoundException e) {
            response.setStatus(404);
        }

        return;
    }

    @DeleteMapping("/delete/{id}")
    @ApiOperation(value = "delete 保存用户",nickname = "User_delete",notes = "删除用户")
    @ResponseStatus(HttpStatus.NO_CONTENT)      //删除成功返回状态码204
    public void delete(@ApiParam(value = "用户") @PathVariable Long id,HttpServletResponse response){
        log.info("---- 删除实体 -----");
        log.info("在数据删除中，可以直接调用M层的delete方法.");
        userService.delete(id);
        return;
    }


    @GetMapping("/getCurrentUserWebAppMenus/")
    @ApiOperation(value = "getCurrentUserWebAppMenus 获取当前用户的菜单权限列表", nickname = "User_getCurrentUserWebAppMenus", notes = "查询前台菜单权限")
    public List<WebAppMenu> getCutUserWebAppMenus() {
        return userService.getCurrentUserWebAppMenus();
    }

    @GetMapping("/checkUsernameIsExist/{username}")
    @ApiOperation(value = "checkUsernameIsExist 检测用户名是否已经存在", nickname = "User_checkUsernameIsExist", notes = "存在：true,不存在:false @author:panjie")
    public boolean checkUsernameIsExist(@ApiParam(value = "用户名（邮箱），必须使用邮箱作为用户名") @PathVariable String username) {
        return userService.checkUsernameIsExist(username);
    }

    @GetMapping("/setUserStatusToNormalById/{id}")
    @ApiOperation(value = "setUserStatusToNormalById 设置某个用户的状态为正常", notes = "author:panjie", nickname = "User_setUserStatusToNormalById")
    public void setUserStatusToNormalById(@ApiParam(value = "用户id") @PathVariable Long id) {
        userService.setUserStatusToNormalById(id);
        return;
    }

    @PostMapping("/register")
    @ApiOperation(value = "register 注册后户", notes = "用户注册时，同时注册部门及用户信息", nickname = "User_register")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void register(@ApiParam(value = "用户注册信息：部门信息+用户信息") @RequestBody DepartmentUser departmentUser) {
        return;
    }


    @ApiModel(value = "部门+用户")
    public static class DepartmentUser {
        @ApiModelProperty("部门")
        private Department department;
        @ApiModelProperty("用户")
        private User user;

        public DepartmentUser() {
        }


        public Department getDepartment() {
            return department;
        }

        public void setDepartment(Department department) {
            this.department = department;
        }

        public User getUser() {
            return user;
        }

        public void setUser(User user) {
            this.user = user;
        }
    }

}
```