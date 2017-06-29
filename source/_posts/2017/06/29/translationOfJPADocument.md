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
```