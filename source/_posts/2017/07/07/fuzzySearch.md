---
title: 模糊查询
date: 2017-07-07 12:48:16
tags: [top10,search,fuzzy]
category: zhangjiahao
---

最近从官方文档学习模糊查询，遇到了一些问题，发现最终还是在对方法做测试的时候出现了很大的问题，不知道该从哪里进行测试。下面以“根据企业类型ID以及企业名称进行contaning模糊查询”为例，进行一下总结。

其实新方法的定义以及使用，主要是分四个步骤：
1 在xxxRepository中进行方法的定义。
2 在xxxService中写接口中方法。
3 在xxxServiceImpl中写接口中方法的实现。
4 在xxxServiceTest中进行该方法的单元测试。

<!--more-->
# 方法定义

最初错误的以为，在方法定义的时候，要使用@Query注解，并且在注解里面将所需要查询的数据写成SQL语句，然后完成对方法的定义。后来发现仅仅需要在xxxRepository中将方法名写好，SpringData就可以自动的根据方法名中的关键字将该方法进行实现，我们只需要在Service中调用该方法就可以实现我们想要的效果。

在ER图中，部门和部门类型有如下关系：
![](/images/er.jpg)

例如，我们想实现通过部门类型id和部门实体内的name字段进行查询，并返回查询到的前10条信息，我们可以如下定义方法：
```
 public interface DepartmentRepository extends CrudRepository<Department, Long> {
      Department findByName(String name);
      Department findOneByPinyin(String pinyin);
      List<Department> findTop10ByDepartmentTypeIdAndNameContaining(Long departmentTypeId, String name);
  }
```

# 接口中的方法

```
public interface DepartmentService {
     List<Department> getTop10ByDepartmentTypeIdAndNameContaining(Long DepartmentTypeId, String name);
}
```

# 接口中方法的实现
```
@Service
public class DepartmentServiceImpl implements DepartmentService {
     @Autowired
     private DepartmentRepository departmentRepository;
 
     @Override
     public List<Department> getTop10ByDepartmentTypeIdAndNameContaining(Long DepartmentTyTypeId, String name) {
        List<Department> departments = departmentRepository.findTop10ByDepartmentTypeIdAndNameContaining(DepartmentTyTypeId, name);
        return departments;
    }
}
```

# 对方法进行单元测试

```
 @SpringBootTest
 @RunWith(SpringRunner.class)
 public class DepartmentServiceTest {
     private static Logger logger = Logger.getLogger(DepartmentTypeServiceTest.class.getName());
     @Autowired
     private DepartmentRepository departmentRepository;
     @Autowired
     private DepartmentService departmentService;
     @Autowired
     private DepartmentTypeRepository departmentTypeRepository;
 
     private DepartmentType departmentType;
     private Set<Department> departments = new HashSet<>();
     private String name;
     @Before
     public void addData() {
         logger.info("创建一个部门类型实体并保存");
         departmentType = new DepartmentType();
         departmentTypeRepository.save(departmentType);
         logger.info("准备测试数据");
         name = CommonService.getRandomStringByLength(32);
         for (Integer i = 0;i < 20; i++)
         {
             Department department = new Department();
             department.setName(i.toString() + name + i.toString());
             department.setCode("15088");
             department.setAddress("天津市北辰区双口镇河北工业大学");
             /**
              * 关键遗漏点：
              * 我们要按照 部门类型id  进行查询，那么我们在保存部门的时候，就需要关联好这个部门类型
              * 排错方向：1.数据是否存储正确。2.是否生成了正确的查询语句。3.实体仓库层的方法名是否正确
              * 排错方法：
              * 1. 删除对ServiceTest的继承，手动写注解，并且不要写 事务 注解
              * 2. logger.info("断言取出了十条数据"); 打断点
              * 3. 使用navicat查看数据表，看是否生成自己想要的信息。
              * 4. 发现未生成department_type_id
              * 5. 得出在新建部门时，没有设置 部门类型 。
              */
             department.setDepartmentType(departmentType);
             departments.add(department);
         }
         departmentRepository.save(departments);
     }
 
     @After public void deleteData() {
         logger.info("删除实体");
         departmentRepository.delete(departments);
         departmentTypeRepository.delete(departmentType);
     }
 
     @Test
     public void getTop10ByDepartmentTypeIdAndName() throws Exception {
         logger.info("查询数据");
         List<Department> list = departmentService.getTop10ByDepartmentTypeIdAndNameContaining(departmentType.getId(), name);
         logger.info("断言取出了十条数据");
         assertThat(list.size()).isEqualTo(10);
     }
} 
```