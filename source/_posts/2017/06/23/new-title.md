---
title: C层自动化文档生成过程
date: 2017-06-23 21:45:17
tags: 
---
# 问题描述
在前后台完全分离的项目中，前后台完成对接主要依据是开发文档，为了保证开发效率，我们引入了自动生成文档的注解，那么它是怎么工作的呢？另外在C层的测试类下，我们需要在文档中生成包含响应信息以及请求信息的文档，那么它们又是如何生成的呢？下面我们以部门类型（DepartmentType）的C层为例，说一下自动化文档生成的过程。
<!-- more -->
# 添加注解
```java
@Api(tags = "DepartmentType (部门类型)", description = "部门 部门类型C层")
@RestController
@RequestMapping("/DepartmentType")
public class DepartmentTypeController {

    @Autowired
    protected DepartmentTypeService departmentTypeService;  // 部门类型

    @GetMapping("/")
    @ApiOperation(value = "获取所有数据", notes = "获取所有数据（未排序 ）@author:panjie", nickname = "DepartmentType_getAll")
    public List<DepartmentType> getAll() {
        return departmentTypeService.findAll();
    }
}
```
在java代码中，如果我们想添加注解，那么首先我们应该引用@Api目录下的各种注解，其中对应于不同的注解有着不同的功能。
例如上例中，@Api(tags = "DepartmentType (部门类型)", description = "部门 部门类型C层") 这一行代码，我们将代码成功提交之后，会发现已经体现在了文档中。你也许会疑惑为什么没有看到nickname的作用呢？在后面我会说一下nickname的作用。
# 效果展示
首先我么会发现在1.5的目录中tags标签里面新增了刚才添加的内容了。如下图所示：

![](/images/tags.png)


另外在第三章中关于各个模块的描述中，我们也同样可以看到新增内容，其中与代码对应情况如下：

![](/images/all.png)

# 生成请求信息和响应信息

我们会发现，除了上面提到的几个API标签对应的内容以外，还有Response等响应信息，那么这些又是怎么生成的呢？这就涉及到我们的ControllerTest类了。
```java
    @Test
    public void getAll() throws Exception 
    {
        logger.info("---- getAll方法测试 ----");
        logger.info("---- 创建一个部门类型实体 ----");
        DepartmentType departmentType = new DepartmentType();
        System.out.println(departmentType);
        logger.info("保存该实体");
        departmentTypeRepository.save(departmentType);
        logger.info("获取响应信息");
        MvcResult mvcResult = this.mockMvc.perform(get("/DepartmentType/")
                .contentType("application/json")
                .header("x-auth-token", xAuthToken))
                .andExpect(status().isOk())
                .andDo(document("DepartmentType_getAll", preprocessResponse(prettyPrint())))
                .andReturn();

        List<DepartmentType> departmentTypeList = (List<DepartmentType>) departmentTypeRepository.findAll();

        logger.info("断言获取到的数据条数与直接使用repository的实体数相同");
        String content = mvcResult.getResponse().getContentAsString();
        JSONArray JSONArray = net.sf.json.JSONArray.fromObject(content);
        assertThat(JSONArray.size()).isEqualTo(departmentTypeList.size());
    }
```
上面代码是C层中getAll方法的单元测试内容，代码this.mockMvc.perform(get("/DepartmentType/")中，get方法内对应的参数，即为Controller中对于get方法定义出来的url，这样子才能用mockMvc向应用程序发出请求，才能够继续执行下面的header、andExpect等函数。其中andDo(document("DepartmentType_getAll", preprocessResponse(prettyPrint())))的作用是获取相应信息并且打印出来，其中document函数中第一个参数，与C层中的nickname相同，这样我们才能正确的获取到打印信息。

单元测试通过以后，我们提交代码并合并到development分支，我们便可以查看到完整的开发文档信息了。☺