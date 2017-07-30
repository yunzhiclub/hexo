---
title: spring中如何用注解实现多个类自动装配同一个接口
date: 2017-07-15 11:32:13
categories: gaoliming
tags: [spring]
---

**转载出处** ：[Spring的注解@Qualifier小结](http://www.cnblogs.com/smileLuckBoy/p/5801678.html) 

<!--more-->
## Example
有如下接口：

```
public interface EmployeeService {
    public EmployeeDto getEmployeeById(Long id);
}
```

同时有下述两个实现类 EmployeeServiceImpl和EmployeeServiceImpl1：

```
@Service("service")
public class EmployeeServiceImpl implements EmployeeService {
    public EmployeeDto getEmployeeById(Long id) {
        return new EmployeeDto();
    }
}

@Service("service1")
public class EmployeeServiceImpl1 implements EmployeeService {
    public EmployeeDto getEmployeeById(Long id) {
        return new EmployeeDto();
    }
}
```

调用代码如下：
```
@Controller
@RequestMapping("/emplayee.do")
public class EmployeeInfoControl {
    
    @Autowired
    EmployeeService employeeService;
     
    @RequestMapping(params = "method=showEmplayeeInfo")
    public void showEmplayeeInfo(HttpServletRequest request, HttpServletResponse response, EmployeeDto dto) {
        #略
    }
}
```
## Error

```
org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'employeeInfoControl': Injection of autowired dependencies failed; nested exception is org.springframework.beans.factory.BeanCreationException: Could not autowire field: com.test.service.EmployeeService com.test.controller.EmployeeInfoControl.employeeService; nested exception is org.springframework.beans.factory.NoSuchBeanDefinitionException: No unique bean of type [com.test.service.EmployeeService] is defined: expected single matching bean but found 2: [service1, service2]
```
### analysis

其实报错信息已经说得很明确了,在autoware时，由于有两个类实现了EmployeeService接口，所以Spring不知道应该绑定哪个实现类，所以抛出了如上错误。


## Solve

用到@Qualifier注解了，qualifier的意思是合格者，通过这个标示，表明了哪个实现类才是我们所需要的，我们修改调用代码，添加@Qualifier注解，需要注意的是@Qualifier的参数名称必须为我们之前定义@Service注解的名称之一

```
@Controller
@RequestMapping("/emplayee.do")
public class EmployeeInfoControl {
    
    @Autowired
    @Qualifier("service")
    EmployeeService employeeService;
    
    @RequestMapping(params = "method=showEmplayeeInfo")
    public void showEmplayeeInfo(HttpServletRequest request, HttpServletResponse response, EmployeeDto dto) {
        #略
    }
}
```