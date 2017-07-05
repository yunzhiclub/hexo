---
title: Spring MVC 使用@JsonView自定义输出字段
date: 2017-06-27 09:01:31
category: teacherPan
tags: ['Spring MVC', 'JsonView']
---
在前后台的分离开发中，我们常常直接输出实体的全部属性。这在大多数情况下都是没有问题的，但有些时候，直接输出全部属性的方法，可能会千万系统死循环的异常，进行影响到数据的输出。本文将阐述使用`@JsonView`注解来避免该问题。

<!--more-->
# 问题描述
比如，我们现在有个树状实体 区域，每个区域都有个父级区域，每个区域又有个子区域集合.
```java
@Entity
@ApiModel(value = "District (区域)", description = "区域")
public class District implements Serializable{
    // 实现了Serializable接口，用于序列化
    private static final long serialVersionUID = 1L;
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    @ApiModelProperty("上级区域")
    @ManyToOne
    private District parentDistrict;
    @OneToMany(mappedBy = "parentDistrict")
    @Lazy
    private List<District> sonDistricts = new ArrayList<>();
```

此时, 当我们查找到某个区域，并将其进行`json`序列化时，将得到一个如下的错误：
```console
Failed to write HTTP message: org.springframework.http.converter.HttpMessageNotWritableException: Could not write JSON document: Infinite recursion (StackOverflowError) (through reference chain: com.mengyunzhi.measurement.repository.District["sonDistricts"]->org.hibernate.collection.internal.PersistentBag[0]->com.mengyunzhi.measurement.repository.District["parentDistrict"]->...
```


# 分析问题
这是由于在进行Json转化时，Spring MVC会自动获取实体上所有属性的值。也就是说`@Lazy`不起作用了。
这直接导致了：
1. 获取某个区域
2. 获取某个区域的父区域
3. 获取父区域的所有子区域
4. 区域子区域的父区域
...

死循环就这么产生了。

# 解决问题
## 使用@JsonIgnore
千万死循环的原因是由于互相调用，所以，我们可以使用`@JsonIgnore`来忽略到任意一方，便可以达到解决问题的目的。但这个方法并不完美。一旦设置了该注解，在前台后分离的架构中，将永远不能够输出该字段的值。这并不是我们想看到的。

## 使用@JsonView
使用`@JsonView`可以对字段输出进行自由的组合
参考官方文档：[https://docs.spring.io/spring/docs/current/spring-framework-reference/html/mvc.html#mvc-ann-jsonview](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/mvc.html#mvc-ann-jsonview)

```
@Entity
@ApiModel(value = "District (区域)", description = "区域")
public class District implements Serializable{
    public interface WithAll {};    
    public interface WithOutParentDistrict  extends WithAll {};
    public interface WithOutSonDistricts extends WithAll {};
    // 实现了Serializable接口，用于序列化
    private static final long serialVersionUID = 1L;
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @JsonView(WithAll.class)
    private Long id;
    @ApiModelProperty("区域名称")
    @JsonView(WithAll.class)
    private String name;
    @ApiModelProperty("上级区域")
    @ManyToOne
    @JsonView(WithOutSonDistricts.class)
    private District parentDistrict;
    @JsonView(WithOutParentDistrict.class)
    @OneToMany(mappedBy = "parentDistrict")
    @Lazy
    private List<District> sonDistricts = new ArrayList<>();
}
```

然后在C层中，将入`@JsonView`以指定该C层，对应输出实体的哪些字段。
```java
    @ApiOperation(value = "get (获取一条数据)", notes = "获取一条数据", nickname = "District_get")
    @GetMapping("/get/{id}")
    @JsonView(District.WithOutSonDistricts.class)
    public District get(@ApiParam(value = "区域实体id") @PathVariable Long id) {
        //获取一条数据
        District district = districtService.get(id);
        return district;
    }
```

此时，在进行json转化时，将不进行子区域的获取。
但新的问题又产生了，那就是我们以后在进行`json`输出时，要必须指定`JsonView`，否则就需要进行所有的转换。这会使得当我们获取一些关联数据时，发生错误。比如我们获取一个部门的时候，将自动获取这个部门对应的区域，那么此时，进行json转换时，还将循环的错误。

## 使用@JsonView并去除@OneToMany
```java
@Entity
@ApiModel(value = "District (区域)", description = "区域")
public class District implements Serializable{
    public interface WithAll {};
    public interface WithOutParentDistrict  extends WithAll {};
    // 实现了Serializable接口，用于序列化
    private static final long serialVersionUID = 1L;
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @JsonView(WithAll.class)
    private Long id;
    @ApiModelProperty("上级区域")
    @ManyToOne
    private District parentDistrict;
    @JsonView(WithOutParentDistrict.class)
    @Transient // 该字段并不存在于数据表中
    private List<District> sonDistricts = new ArrayList<>();
```
此时，我们便实现了：
1. 在不加`@JsonView`注解的情况下，由于`@Transient`的存在，使得`sonDistricts`保持为空集合。
2. 在加入`@JsonView(District.WithOutParentDistrict.class)`的情况下，将输出`sonDistricts`同时，忽略输出`parentDistrict`.
3. 使用的接口继承，便于代码在后期的扩展。

为了上述实体的工作，我们还需要手动的去获取子区域。
```java
public interface DistrictRepository extends CrudRepository<District, Long> {
    District getByName(String name);
    List<District> getAllByParentDistrictId(Long id);
}
```

```java
    @Override
    public District getTreeByDistrict(District district) {
        List<District> districts = districtRepository.getAllByParentDistrictId(district.getId());
        for(District district1 : districts) {
            this.getTreeByDistrict(district1);
        }
        district.setSonDistricts(districts);
        return district;
    }
```

C层使用方法：
```java
    @ApiOperation(value = "getTreeById", notes = "获取当前用户所有的部门树", nickname = "District_getTreeById")
    @GetMapping("/getTreeById/{id}")
    @JsonView(District.WithOutParentDistrict.class)
    public District getTreeById(@ApiParam(value = "区域ID") @PathVariable Long id) {
        District district = districtService.get(id);
        districtService.getTreeByDistrict(district);
        return district;
    }
```

# 总结
由于jackjson在进行json序列化时，将默认序列化所有的字段。所以，我们需要使用`@JsonIgnore`或`@JsonProperty`来排除某些字段。但一味的全部排除有时又不能满足我们的需求。这时候，就需要使用定制性更强的`@JsonView`。
使用`@JsonView`虽然能够解决特殊的需求，但是如果由于实体间的关联关系。每进行一次输出，都定制一次`@JsonView`又过于麻烦。最终在实际的问题上，我们删除了`OneToMany`的关系，手动的维护了`OneToMany`的关系。这使我们只在进行特定输出时使用`@JsonView`来解决问题。

除了在实体中，直接建立接口以外，我们也可以单独的新建一个类，然后在这个类中，建立接口，或是抽象类。并将接口或是抽象类应用到实体的字段中。

在特殊的时候，以上的方法可以还解决不了我们的问题。如果是这样，我们就需要单独的建立用于进行`json`序列化输出的类了。然后在这个类中，去定义`get`函数，`spring mvc`，则会自动触发`get`中的方法，进而进行json的自动转换.


参考文档：
[https://docs.spring.io/spring/docs/current/spring-framework-reference/html/mvc.html#mvc-ann-jsonview](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/mvc.html#mvc-ann-jsonview)

代码示例：



