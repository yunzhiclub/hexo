---
title: JPA参考文献的翻译（部分）
date: 2017-06-29 20:06:12
tags: [JPA,documents,translation]
category: zhangjiahao
---
# 4.使用Spring数据仓库

抽象的说，Spring数据仓库()目标是显著的（significantly）减少为各种持久化存储实现数据访问所需要的示例代码的数量。

> spring数据存储仓库文档和模块
> 这一章节解释Spring数据仓库的核心概念和接口，本章节的信息是从（pull from）Spring Data Common模块中提取的。它使用了JPA模块的配置（configuration）和代码实例。适应了XML声明的命名空间和类型，并且扩展了你正在使用的特定模块的类型的等价物。命名空间的引用（reference）涵盖了支持存储库API的所有Spring数据模块支持的XML配置，存储库查询关键字涵盖了通常由存储库抽象支持的查询方法关键字。有关模块的具体特性的详细信息，请参阅本文档模块的章节。

## 4.1核心概念

Spring数据仓库（Spring Data Repository）的中心接口是仓库（Repository）。它需要域类来管理，并且需要域类的id类型作为参数（argument）类型。接口起的主要作用就是作为一个标记接口去捕获要处理的类型并且帮助你去发现该接口的拓展接口。CrudRepository为正在被管理的实体类提供了复杂的（sophisticated）增删改查方法。

> 示例3. CrudRepository接口

```
public interface CrudRepository<T, ID extends Serializable>
    extends Repository<T, ID> 
{
    <S extends T> S save(S entity); //保存给出的实体
    T findOne(ID primaryKey);       //通过给出的id返回对应的实体
    Iterable<T> findAll();          //返回所有的实体
    Long count();                   //返回实体的数量
    void delete(T entity);          //删除给出的实体
    boolean exists(ID primaryKey);  //表示给出id的实体是否存在
    // … more functionality omitted.//更多的方法省略了(omit)
}
```

> 我们也提供了持久特定于技术（technology-specific）的抽象例如：JpaRepository和MongoRepository。这些接口继承了CrudRepository并且暴露了底层的持久化技术的功能（capabilities）除了一些一般（generic）的持久性技术无关的接口，如:CrudRepository。

在CrudRepository之上这里还有一个抽象的PagingAndSortingRepository，它添加了额外的方法来简化对实体的分页访问（paginated access）。

> 示例4. PageAndSortingRepository

```
public interface PagingAndSortingRepository<T, ID extends Serializable>
  extends CrudRepository<T, ID> {
  Iterable<T> findAll(Sort sort);
  Page<T> findAll(Pageable pageable);
}
```

如果你想访问第二页的User并且分页大小是20，你可以简单的这样做：

```
PagingAndSortingRepository<User, Long> repository = // … get access to a bean
Page<User> users = repository.findAll(new PageRequest(1, 20));
```

除了查询方法以外，还可以查询计数和删除查询的查询派生也是可以用到的。
> 示例5. 推导出查询的总数

```
public interface UserRepository extends CrudRepository<User, Long> {
  Long countByLastname(String lastname);
}
```

> 示例6. 推导出删除的总数

```
public interface UserRepository extends CrudRepository<User, Long> {
  Long deleteByLastname(String lastname);
  List<User> removeByLastname(String lastname);
}

## 4.2 查询方式

标准的增删改查功能（functionally）仓库对底层（underlying）的数据存储库（datastore）有查询操作。通过Spring Data,声明这些查询分以下四步：

1. 声明一个借口继承Repository或者Repository的某一个子接口（subinterface）并且将它输入到它要处理的域类（domain class）和ID类型。
```
interface PersonRepository extends Repository<Person, Long> { … }
```

2. 在这个接口声明查询方式
```
interface PersonRepository extends Repository<Person, Long> {
  List<Person> findByLastname(String lastname);
}
```

3. 设置（set up）Spring为这些接口创建代理实例（proxy instance），通过JavaConfig:

```
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
@EnableJpaRepositories
class Config {}
```

或者通过XML configuration:
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xmlns:jpa="http://www.springframework.org/schema/data/jpa"
   xsi:schemaLocation="http://www.springframework.org/schema/beans
     http://www.springframework.org/schema/beans/spring-beans.xsd
     http://www.springframework.org/schema/data/jpa
     http://www.springframework.org/schema/data/jpa/spring-jpa.xsd">
   <jpa:repositories base-package="com.acme.repositories"/>
</beans>
```

在本例中使用了JPA的命名空间，如果你正在为其他存储库使用存储仓库抽象，你需要为你的存储模块改变到一个合适的声明的命名空间。您需要将其更改为您的存储模块的适当名称空间声明，该声明应该有利于（in favor of）交换jpa，例如mongodb。
另外，请注意，JavaConfig变量（variant）不会在默认情况下使用带注释的类的包。要定制（customize）这个包，可以扫描使用一个basePackage.数据存储特定存储库的属性@enable注解。

4. 获取注入（inject）的repository实例（instance）且使用它。

```
public class SomeClient {
  @Autowired
  private PersonRepository repository;
  public void doSomething() {
    List<Person> persons = repository.findByLastname("Matthews");
  }
}
```

接下来的章节将详细的解释上面的每一步。

## 4.3定义repository接口

第一步你应该定义一个域类特定（specific）的repository的接口。这个接口必须继承Repository并且输入到域类和ID类型中。如果你想为这个域类型公开（expose）CRUD方法，继承CrudRepository而不是继承Repository。

### 4.3.1 微调库（Fine-tuning）定义

通常，你的Repository接口将会继承Repository,CrudRepository或者PagingAndSortingRepository。或者（Alternatively），如果你想继承Spring Data的接口，你也可以用@RepositoryDefinition注解你的repository。继承CrudRepository公开了一系列完整的方法来操作（manipulate）你的实体。如果你想选择这些公开的方法，只需要将这些公开的方法复制的进你的域存储库（domain repository）中。
> 这将会允许你在提供的Spring数据存储库功能之上定义你自己的抽象。

示例7 选择公开的CRUD方法
```
@NoRepositoryBean
interface MyBaseRepository<T, ID extends Serializable> extends Repository<T, ID> {
  T findOne(ID id);
  T save(T entity);
}
interface UserRepository extends MyBaseRepository<User, Long> {
  User findByEmailAddress(EmailAddress emailAddress);
}
```

在第一步中你可以为你的所有的域存储库（domain repository）定义一个公有的基础接口（base interface）并且公开findOne()和save()等方法。这些方法将会被路由到（routed into）Spring Data提供的你选择的基本的存储repository实现。例如：
