---
title: 解决使用spring boot mvc的@JsonView注解返回page为空的问题
date: 2017-07-30 11:43:02
category: teacherPan
tags: ['spring boot', '@JsonView', 'spring mvc']
---
要`spring mvc`中，我们使用`@JsonView`注解来定义内容输出。但如果我们返回分页数据`Page`的话，却意外的行到了`{}`。这是由于`@JsonView`注解并没有在`Page`类型上起作用。所以导致返回类型为`Page`时，输出空内容。

<!-- more -->
# 解决方案
针对此问题，网上的解决方案并不统一。最后，找到如下解决方案: 即对`WebMvcConfigurerAdapter`进行配置。重写`public void configureMessageConverters`方法。

```java
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {
    static private Logger logger = Logger.getLogger(WebConfig.class.getName());
    /**
     * 解决spring mvc @jsonview page empty(springMvc 使用jsonView时，返回page信息为空对象)的问题
     * 这个问题的解决方案适用的比较少，而且还有信息存在误导。
     * 最后总结下解决方法：
     * 1.绝对不能在配置文件中去配置那个 DEFAULT_VIEW_INCLUSION。
     * 如果这么做了，就会发现C层配置的JsonView会完全失去作用
     * 2. 本问题中，spring 官方社区的回答比stackoverflow 还要靠谱。
     * 在 https://jira.spring.io/browse/SPR-13331 的回答中，甚至有了给出了github的地址
     * 3. 参考代码时，要使其它的配置完全相同，比如我在参考时，就是在已经设置了DEFAULT_VIEW_INCLUSION为true的前提下；
     *    而这个配置项，参考代码中并没有。进而没有在第一时间内验证出参考代码的正确性。
     * 正确的可被参考的代码：https://github.com/sdeleuze/SpringSampleProject
     * 
     * 总结：问题解决了大概两天的时间。开始低估了问题了难度，当发现初步找的答案并不能解决实际问题时。
     * 并没有沉下心来去看每一篇google到的文章。
     * 最后，静心的仔细的学习了每一篇文章，并找到了相关的示例代码，同时删除了按照其它方法配置的不适用配置项后。
     * 问题得以解决。
     * 所以：
     * 1. 当发现低估问题的难度的时候，确诊关键字没有问题，就要静心来学习。
     * 2. 每找到一个方法，都值得一试。但不要将两种解决方案同时试。试第一种不行，就要将代码进行恢复。
     * 然后才能继续试第二种。
     * 3. 读懂找到资料的每一行英文，这很重要！
     * @param converters
     */
    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        ObjectMapper mapper = Jackson2ObjectMapperBuilder.json().defaultViewInclusion(true).build();
        converters.add(new MappingJackson2HttpMessageConverter(mapper));
    }
}
```
# 使用方法
在使用时，我们可以直接使用`@JsonView`在控制器上，如果我们未设置实体中的注解信息，则会发现什么变化都没有发生。
如果我们想排除某此字段，则需要如下使用：

假设我们有以下实体：
```java
// 学生
class Student {
    private String name;
    private boolean sex;
    @ManyToOne
    private Klass klass;    // 班级
}
```

```java
// 班级
Class Klass {
    private String name;
    @OneToMany(mappedBy = "klass")
    private Set<Student> students = new HashSet<>();  
}
```
## 场景一
在输出学生时，输出班级信息，同时忽略班级中的`students`
控制器如下：
```java
@RestController
@RequestMapping("/Student")
Class StudentController {
    @GetMapping(/{id})
    public Student get(@PathVar....) {
        ....
    }
}
```

1. 建立一个接口，并用这个接口注解`@ManyToOne`, `@OneToMany`.
```java
interface NoneJsonView {}
```
加入注解，只有在控制器中使用`NoneJsonView`时，才会被序列化.
```java
// 学生
class Student {
    private String name;
    private boolean sex;
    @ManyToOne
    @JsonView(NoneJsonView.class)
    private Klass klass;    // 班级
}
```

```java
// 班级
Class Klass {
    private String name;
    @OneToMany(mappedBy = "klass")
    @JsonView(NoneJsonView.class)
    private Set<Student> students = new HashSet<>();  
}
```

2. 在建立一个接口，用以输出指定字段
```java
interface StudentGet {}
```
由于我们想输出学生的班级信息，所以将注解加入到学生的班级字段上。

```java
// 学生
class Student {
    private String name;
    private boolean sex;
    @ManyToOne
    @JsonView({NoneJsonView.class, StudentGet.class})
    private Klass klass;    // 班级
}
```

3. 在控制器上加入注解
```java
@RestController
@RequestMapping("/Student")
Class StudentController {
    @GetMapping(/{id})
    @JsonView(StudentGet.class)
    public Student get(@PathVar....) {
        ....
    }
}
```

我们在控制器上使用的注解为`@JsonView(StudentGet.class)`表示：只有未加注解或是加了注解并且对应`@JsonView(StudentGet.class)`的字段，才会被序列化。
由于`Stuent`的全部字段都符合上述的规则，所以全部会被序列化。
而`Klass`中的`stuents`字段，由于加入了`@JsonView(NoneJsonView.class)`，但该注解并不对应`@JsonView(StudentGet.class)`, 所以将不被序列化。

目标实现

## 场景二
输出某个班级时，对应输出其班级上所有的学生，但自动忽略学生上的班级信息。

```java
@RestController
@RequestMapping("/Klass")
Class KlassController {
    @GetMapping(/{id})
    public Klass get(@PathVar....) {
        ....
    }
}
```

1. 新建注解，并且作用于班级的students中.
```java
interface KlassGet{}
```

```java
// 班级
Class Klass {
    private String name;
    @OneToMany(mappedBy = "klass")
    @JsonView({NoneJsonView.class, KlassGet.calss})
    private Set<Student> students = new HashSet<>();  
}
```

2. 在控制器上加入注解
```java
@RestController
@RequestMapping("/Klass")
Class KlassController {
    @GetMapping(/{id})
    @JsonView(KlassGet.calss)
    public Klass get(@PathVar....) {
        ....
    }
}
```

此时，由于`Student`的`klass`字段，拥有注解。并且拥有的注解，不包含`@JsonView(KlassGet.calss)`，所以在序列化时，将被自动省略.

## 深入理解
加入上述配置前，使用`@JsonView`注解时，如果字段并未设置`@JsonView`注解。那么则默认 **不进行** 序列化。
加入上述配置后，我个人的理解是：如果字段并未设置`@JsonView`注解。那么则默认 **进行** 序列化。
加入配置的前后，对于加入`@JsonView`注解的字段，判断方法相同：符合，则序列化；不符合，不序列化。

这样，使用上述方法便解释了为什么加入配置后，`Page`类型便得以全部输出了。
这时由于`Page`并没有加入`@JsonView`，所以会全部自动序列化。

在实际的使用过程中，这种思想（只有加了`@JsonView`才可能不被序列化 ）结合`@JsonView`的使用，大幅的缩减注解代码量。

# 总结：
我们将`@JsonView`注解的默认规则由：
只有加入了注解才可能 **被** 序列化 
修改为:
-> 只有加入了注解才可能 **不被** 序列化。
修改默认规则前：`Page`并没有加入`@JsonView`注解，默认忽略。
修改默认规则后：`Page`并没有加入`@JsonView`注解，默认自动转换。
同时，也解决了使用`@JsonView`注解时，需要大量加入注解代码的尴尬。
