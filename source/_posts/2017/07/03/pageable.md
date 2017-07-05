---
title: 如何实现分页（M层）
date: 2017-07-03 20:35:42
tags: [pageable, PagingAndSortingRepository]
category: zhangjiahao
---

今天学习了潘老师写的器具类别的分页，讲述一下对代码的理解。有不足之处，欢迎批评指正。

# 将获取到的数据进行规定条数的分页

## 继承PagingAndSortingRepository

首先我们对获取到的所有的数据进行分页。我们应该知道，分页并不是CrudRepository里面自带的方法，所以我们需要引入。简单的方法是，我们可以在InstrumentTypeRepository中，将其进行对PagingAndSortingRepository的继承，然后在该类定义函数如下：

```
public interface InstrumentTypeRepository extends PagingAndSortingRepository<InstrumentType, Long>
{

}
```
<!-- more -->
## M层直接调用方法
这样，在InstrumentTypeServiceImpl类中就可以调用PagingAndSortingRepository中的分页方法了。下面是对分页数为20的代码的编写：

我们在InstrumentTypeServiceImpl中关于getAll的方法如下所示：
```
public List<InstrumentType> getAll()
{
	List<InstrumentType> list = new ArrayList<InstrumentType>();
	list = (List<InstrumentType>) instrumentTypeRepository.findAll(new PageRequest(page:1, size:20));
	return list;
}
```

可是我们很轻易的就发现，这样写虽然很简单，但是不能动态的生成我们需要的分页数。假如我们想生成分页数为10的话，那么还得需要我们去手动的修改源代码。所以接下来我们看一看如何动态的生成分页的代码。

# 动态生成分页

## Repository中进行方法的定义

同样的，我们还是以InstrumentType为例。示例方法为通过学科id获取对应的全部的器具类别信息并进行分页。但是这次我们像往常一样只需要继承CrudRepository。
```
public interface InstrumentTypeRepository extends CrudRepository<InstrumentType, Long>{
    Page<InstrumentType> findAllByDisciplineId(Long id, Pageable pageable);
}
```

我们看到在该类下定义了函数findAllByDisciplineId，其中传入的参数为学科id，和pageable对象。pageable需要进行Pageable类的引入，同样的Page类也需要进行引入，代码如下：
```
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
```
当然，如果我们用的IDEA编译器的话，它会自动提示我们去引入该包。
> Pageable是Spring Data库中定义的一个接口，该接口是所有分页相关信息的一个抽象，通过该接口，我们可以得到和分页相关的所有信息，例如pageNumber、pageSize等。
Pageable定义了很多方法，但是我们需要用到的信息只有两个，一个是分页的信息（page、size），一个是排序的信息。在springmc的请求中只需要在方法的参数中直接定义一个pageable类型的对象，当Spring发现这个参数时，Spring会自动根据request的参数来组装该pageable对象，Spring支持的request参数如下：
```
    page，第几页，从0开始，默认为第0页  
    size，每一页的大小，默认为20  
    sort，排序相关的信息，以property,property(,ASC|DESC)的方式组织，例如sort=firstname&sort=lastname,desc表示在按firstname正序排列基础上按lastname倒序排列。  
```
> 不太明白的一点是，findAllByDisciplineId这个方法名是自己起的，但是起名的格式该类下在有没有什么规定，才能实现通过学科id获取器具类别的信息。

## Service层以及Service层的实现类
然后再Service层我们定义分页的方法：
```
Page<InstrumentType> getAllByDisciplineId(Long id, Pageable pageable);
```
其中返回值为page类型的数组对象，传入值为学科id以及pageable对象。

ServiceImpl中,我们进行方法的实现：
```
@Override
    public Page<InstrumentType> getAllByDisciplineId(Long id, Pageable pageable) 
    {
        Page<InstrumentType> page = instrumentTypeRepository.findAllByDisciplineId(id, pageable);
        return page;
    }
```
在实现类中，我们将传入的pageable对象利用Repository类中定义的方法，返回值为类型为Page的数组对象。

## Test类中的测试

有了定义好的方法，下面我们对其进行测试。
```
	@Test
    public void page() {
    	// 新建学科实体，设置name、pinyin属性并保存
        Discipline discipline = new Discipline();
        discipline.setName("namesdfsdfdf");
        discipline.setPinyin("pinyin");
        disciplineRepository.save(discipline);
		// 新建器具类别实体，设置name、pinyin、学科类别属性并保存
        InstrumentType instrumentType = new InstrumentType();
        instrumentType.setName("name");
        instrumentType.setPinyin("pinyin");
        instrumentType.setDiscipline(discipline);
        instrumentTypeRepository.save(instrumentType);
		// 新建器具类别实体1，设置name、pinyin、学科类别属性并保存
        InstrumentType instrumentType1 = new InstrumentType();
        instrumentType1.setName("namesdfd");
        instrumentType1.setPinyin("pinyinsdfsdf");
        instrumentType1.setDiscipline(discipline);
        instrumentTypeRepository.save(instrumentType1);		
		// 定义pageRequest对象，设置分页信息。从第一页开始显示，分页大小为1。
        final PageRequest pageRequest = new PageRequest(
                page:1, size:1
        );
        Page<InstrumentType> page = instrumentTypeRepository.findAllByDisciplineId(discipline.getId(), pageRequest);
        // 断言一共是两页，因为分页大小为1，有两条信息
        assertThat(page.getTotalPages()).isEqualTo(2);
        // 断言分页大小为1
        assertThat(page.getContent().size()).isEqualTo(1);
        // 删除器具类别实体，删除学科类别试题
        instrumentTypeRepository.delete(instrumentType1);
        instrumentTypeRepository.delete(instrumentType);
        disciplineRepository.delete(discipline);
        return;
    }
```

这样我们就可以在C层去调用刚刚写好的关于分页的函数啦！