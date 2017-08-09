---
title: 学入学习JPA的@ManyToMany注解 study jpa @ManyToMany deeply
date: 2017-08-09 17:40:29
tags: ['jpa', '@ManyToMany']
category: teacherPan
---
在JPA中，实间的关系属`@ManyToMany`最难理解，使用起来最为复杂。当存在某个实体的关键字由多个属性组成时，复杂程序就更大了。借此， 我们以以下ER图为例，对`@ManyToMany`进行一次深入学习。

ER图如下：
{% asset_img 1.png ER图 %}
<!-- more  -->
# 基础代码

## 标准装置实体
普通的实体，代码如下：
```java
 */
@Entity @ApiModel(value = "DeviceSet (计量标准装置)", description = "计量标准装置实体")
public class DeviceSet implements Serializable{
    private static final long serialVersionUID = 1L;
    @Id @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    // setter/getter
}
```

## 授权检定项目实体
该实体的主键由两个外键组成(如何使用组合键，并不属于本文的讨论范围)
```java
@Entity
@Immutable
@ApiModel(value = "DeviceInstrument (装置授权检定项目)", description = "装置授权检定项目实体")
public class DeviceInstrument {
    @Embeddable
    public static class Id implements Serializable {        //封装组合健
        @Column(name = "accuracy_id", nullable = false, insertable = false, updatable = false)
        private Long accuracyId;
        @Column(name = "measure_scale_id", nullable = false, insertable = false, updatable = false)
        private Long measureScaleId;

        public Id() {
        }

        public Id(Long accuracyId, Long measureScaleId) {
            this.accuracyId = accuracyId;
            this.measureScaleId = measureScaleId;
        }

        public Long getMeasureScaleId() {
            return measureScaleId;
        }

        public void setMeasureScaleId(Long measureScaleId) {
            this.measureScaleId = measureScaleId;
        }

        public Long getAccuracyId() {
            return accuracyId;
        }

        public void setAccuracyId(Long accuracyId) {
            this.accuracyId = accuracyId;
        }

        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (!(o instanceof Id)) return false;

            Id id = (Id) o;

            if (!measureScaleId.equals(id.measureScaleId)) return false;
            return accuracyId.equals(id.accuracyId);
        }

        @Override
        public int hashCode() {
            int result = measureScaleId.hashCode();
            result = 31 * result + accuracyId.hashCode();
            return result;
        }
    }

    @EmbeddedId                 //映射组合建
    private Id id = new Id();

    @ManyToOne
    @ApiModelProperty("对应的测量范围")        //关联测量范围
    @JoinColumn(insertable = false, updatable = false)
    private MeasureScale measureScale;

    @ManyToOne
    @ApiModelProperty("对应的精度")         //关联精度
    @JoinColumn(insertable = false, updatable = false)
    private Accuracy accuracy;

    // 其它set/get代码省略，下同
}
```

# 默认@ManyToMany
## 标准装置实体
我们在该实体中加入多对多的字段，查看其为我们生成的默认字段：
```java
@Entity @ApiModel(value = "DeviceSet (计量标准装置)", description = "计量标准装置实体")
public class DeviceSet implements Serializable{
    private static final long serialVersionUID = 1L;
    @Id @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    @ManyToMany
    @ApiModelProperty("对应的装置授权检定项目")
    private Set<DeviceInstrument> deviceInstruments = new HashSet<DeviceInstrument>();
```

进行单元测试：
{% asset_img 2.png 表基本结构 %}

{% asset_img 1.gif 表主键及外键 %}

最终，我们得出以下结论：
> 当一个普通实体与另一个组合主键实体进行多对多关联系时:

1. 为了保存多对多关系，JPA为我们自动创建一个中间表。表名为：`本表名_关联表名s`
2. 此表为我们生成了三个字段，三个字段均为主键。
3. 其中一个主键与 `普通实体` 主键相关联，命名为：`表名_主键名`
4. 另两个主键组合在一起，与 `组合主键实体` 的组合主键相关联。命名为：`表名s_主键名`

## 授权检定项目实体
同样，我们加入默认的多对多注解，不做任何设置

```java
@Entity
@Immutable
@ApiModel(value = "DeviceInstrument (装置授权检定项目)", description = "装置授权检定项目实体")
public class DeviceInstrument {
    //参考网址http://codego.net/144328/



    @EmbeddedId                 //映射组合建
    private Id id = new Id();

    @ManyToOne
    @ApiModelProperty("对应的测量范围")        //关联测量范围
    @JoinColumn(insertable = false, updatable = false)
    private MeasureScale measureScale;

    @ManyToOne
    @ApiModelProperty("对应的精度")         //关联精度
    @JoinColumn(insertable = false, updatable = false)
    private Accuracy accuracy;

    @ManyToMany
    private Set<DeviceSet> deviceSets;

}
```
同样，任意进行单元测试，重新生成数据表：
{% asset_img 3.png 表基本结构 %}

{% asset_img 2.gif 表主键及外键 %}

最终，我们得出以下结论：
> 当一个组合主键实体与另一个普通实体进行多对多关联系时:

1. 为了保存多对多关系，JPA为我们自动创建一个中间表。表名为：`本表名_关联表名s`
2. 此表为我们生成了三个字段，三个字段均为主键
3. 其中一个主键与 `普通实体` 主键相关联，命名为：`表名s_主键名`
4. 另两个主键组合在一起，与 `组合主键实体` 的组合主键相关联。命名为：`表名_主键名`

我们将以上规律进行总结后不难得出：
1. 为了保存多对多关系，JPA为我们自动创建一个中间表。表名为：本表名_关联表名s
2. 此表为我们生成了三个字段，三个字段均为主键
3. 其中一组主键（可能是一个，也可以是多个）与本实体主键相关联，命名为：`表名_主键名0`, `表名_主键名1`
4. 另一组主键(可能是一个，也可以是多个)关联实体相关联，命名为：`关联实体表名s_主键名0`, `关联实体表名s_主键名1`


# 自定义多对多
两个`@ManyToMany`虽然已经生效，但是却生成了两张数据表，而且这两张表对应的字段也不一样，那么如何使用一个数据表来存储两个实体的`@ManyToMany`关系呢？

## 统一字段
两个表合一的前提是字段名统一，由于上述第4点原因，导致关联实体的字段名中有个多余的`s`，下面，我们使用注解来自定义自段名称.

### 标准装置实体

```java
public class DeviceSet implements Serializable{
    @ManyToMany
    @ApiModelProperty("对应的装置授权检定项目")
    @JoinTable(
        joinColumns = {@JoinColumn(name = "device_set_id", referencedColumnName = "id")}
        inverseJoinColumns = {
                @JoinColumn(name = "accuracy_id", referencedColumnName = "accuracy_id"),
                @JoinColumn(name = "measure_scale_id", referencedColumnName = "measure_scale_id")
        }
    )
    private Set<DeviceInstrument> deviceInstruments = new HashSet<DeviceInstrument>();
```
`@JoinTable`有两个(不止这两个)重要的属性：`joinColumns`(关联列）和`inverseJoinColumns`(反关联列), 在这里，它们的主体都是系统为我们生成的中间表。
`joinColumns = {@JoinColumn(name = "device_set_id", referencedColumnName = "id")}`译为：
关联列属性：本表（系统生成的中间表）中的`device_set_id`关联当前实体的`id`字段。
<hr>
```java
inverseJoinColumns = {
        @JoinColumn(name = "accuracy_id", referencedColumnName = "accuracy_id"),
        @JoinColumn(name = "measure_scale_id", referencedColumnName = "measure_scale_id")
}
```
译为：

反（上面自动关系的当前实体，这个“反”字，代表关联ToMany的另一个实体）关联列属性： 1. 本表（系统生成的中间表）中的`accuracy_id`字段，关联另一个实体的`accuracy_id`字段; 2.  本表（系统生成的中间表）中的`measure_scale_id`字段，关联另一个实体（授权检定项目）的`measure_scale_id`字段;

我们启动单元测试：
{% asset_img 4.png 表基本结构 %}

### 授权检定项目实体
有了前面的经验，下面就很简单了：

```
public class DeviceInstrument {
    @ManyToMany
    @JoinTable(
            joinColumns = {
                    @JoinColumn(name = "accuracy_id", referencedColumnName = "accuracy_id"),
                    @JoinColumn(name = "measure_scale_id", referencedColumnName = "measure_scale_id"),
            },
            inverseJoinColumns = {@JoinColumn(name = "device_set_id", referencedColumnName = "id")})
    private Set<DeviceSet> deviceSets;
```
同样，我们翻译一下：
joinColumns: 中间表字段关联属性：1. 本中间表的accuracy_id对应当前实体的accuracy_id; 2.本中间表的measure_scale_id对应当前实体的measure_scale_id
inverseJoinColumns: 中间表字段关联对方实体属性: 1. 本中间表的device_set_id字段，对应对方实体(标准装置)id字段。

单元测试略

## 统一表名
最后，我们统一使用`@JoinTable`的`name`属性，来统一表名，将以前系统合并的两张表，用一张表来表示：

```java
    @ManyToMany
    @JoinTable(
            name = "device_set_device_instruments",
            joinColumns = {
                    @JoinColumn(name = "accuracy_id", referencedColumnName = "accuracy_id"),
                    @JoinColumn(name = "measure_scale_id", referencedColumnName = "measure_scale_id"),
            },
            inverseJoinColumns = {@JoinColumn(name = "device_set_id", referencedColumnName = "id")})
    private Set<DeviceSet> deviceSets;
```

```java
    @ManyToMany
    @ApiModelProperty("对应的装置授权检定项目")
    @JoinTable(
            name = "device_set_device_instruments",
            joinColumns = {@JoinColumn(name = "device_set_id", referencedColumnName = "id")},
            inverseJoinColumns = {
                    @JoinColumn(name = "accuracy_id", referencedColumnName = "accuracy_id"),
                    @JoinColumn(name = "measure_scale_id", referencedColumnName = "measure_scale_id")
            }
    )
    private Set<DeviceInstrument> deviceInstruments = new HashSet<DeviceInstrument>();
```

# 总结：
一直在多对多的问题上一知半解，使用的注解大多也来源于课本和网络。但最终掌握该问题最好最快最牢的方法仍然是“实战”。没错，通过半小时的实战，我们一点点的用代码测试出了`@JoinTable`的几个核心注解的功能。当以后再出现多对多注解时，我们知道：
1.当前注解的对象为：中间表。
2.关联的字段与反关联的字段，分别是指当前实体与对方实体。
3.关联的表名，字段名都可以自定义。
4.关联的字段，本来就是一个数组`{}`，这个数组中可以有一个参数，也可以有多个参数。

