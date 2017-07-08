---
title: hibernate级联操作
date: 2017-07-06 11:13:33
tags: [cascade, hibernate]
categories: gaoliming
---
当我们处理实体之间关系的时候，很可能会涉及到一个实体发生改变的时候它所关联的实体也会发生改变，这就是我们所说的级联操作。当然，hibernate已经给我们设计好了我们所要用的简单的级联操作。**详细的代码见 参考文章 的链接**
<!--more-->
## 级联保存
### 关联关系中比较常用：@OneToMany, @ManyToMany, @OneToOne
对于级联保存而言，是最基本的级联关系，估计也是我们最常用的关系。也就是保存本实体的时候，它所关联的实体也会自动保存。级联保存操作为``CascadeType.PERSIST``

## 级联更新
### 关联关系中比较常用：所有实体关系都适用
对于更新，当然也是我们最常见的情况。我们说级联更新基本上适用于所有的情况。但是对于``@OneToMany``的情况下，我们可能需要结合下面所讲到的**孤立移除**才能够完善。级联更新操作为``CascadeType.MERGE``

## 级联删除
### 关联关系中比较常用：@OneToMany， @OneToOne
我们可以这样想，当一个商品可以有多个价格，但是当我们从数据库中删除这个商品的时候，我们说这个商品所对应的价格也就没有任何意义了。所以要删除一个商品的正确顺序是先删除这个商品对应的价格然后删除这个商品，如果不这样的话删除商品的时候会被**外健**约束从而删除商品失败。而hibernate对应的就是级联删除则是``CascadeType.REMOVE``
## 孤立移除
### 适用于：@OneToMany， @ManyToMany
例如：
![](/images/onetomany.png)
我们在器具类别实体中关联着多个规格型号，例如有规格型号1,规格型号2.这时我们更新的时候想要删除规格型号1,这时候我们就需要``orphanRemoval = true``来实现``孤立移除``,这样就可以孤立的删除规格型号1.具体代码如下：
```
/**
 * 器具类别  实体
 */
@Entity
@ApiModel(value = "InstrumentType (器具类别)", description = "器具类别实体")
public class InstrumentType implements Serializable {
    private static final long serialVersionUID = 1L;
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    @ApiModelProperty("名称")
    @Column(nullable = false)
    private String name;
    @Column(nullable = false)
    @ApiModelProperty("拼音")
    private String pinyin;

    @Lazy       //参照hibernate实战第七章第三节级联状态
    @OneToMany(mappedBy = "instrumentType", cascade = {CascadeType.PERSIST, CascadeType.REMOVE, CascadeType.MERGE}, orphanRemoval = true)
    @ApiModelProperty("oneToMany 规格型号")
    private List<Specification> specifications = new ArrayList<>();
   // getter and setter ....
}
```
## 参考文章
hibernate第七章第三节级联状态
[Hibernate@ManyToMany总结](https://gaoliming123.github.io/2017/06/06/ManyToMany/) 