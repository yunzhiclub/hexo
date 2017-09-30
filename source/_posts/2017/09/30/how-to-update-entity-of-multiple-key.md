---
title: 多关键字实体的更新方法
date: 2017-09-30 08:31:44
tags: 
    - SpringMVC
    - JPA
category: teacherPan
---
假设我们有以下实体，该实体共有4个属性，其中有2个属性组成了组合主键。

{% asset_img 1.png ER图 %}

下面，简单阐述下如何正确的实现对上述实体类型的更新操作。
<!--more-->
# 操作习惯
在进行更新时，按以往的方法，我们只需要设置好这个实体的主键，然后调用`save`方法，就可以了。如下:
```java
    @Override
    public void saveOrUpdate(DeviceInstrument deviceInstrument) {
        deviceInstrumentRepository.save(deviceInstrument);
        return;
    }
```
结果：失败。

# 有则更新，无则插入
那么我们换种思路：
造成前面的方法失效的原因：可能是`SpringMVC`在进行`save`操作时，不能够区分是插入还是更新。

解决方法：在进行数据保存前，先查找是否有历史记录。如果有，执行更新操作；没有，则执行插入操作。

代码如下：

```java
    @Override
    public void saveOrUpdate(DeviceInstrument deviceInstrument) {
        DeviceInstrument oldDeviceInstrument = deviceInstrumentRepository.findOne(deviceInstrument.getId());
        if (null == oldDeviceInstrument) {
            logger.info("新记录，保存");
            deviceInstrumentRepository.save(deviceInstrument);
        } else {
            logger.info("老记录，更新");
            oldDeviceInstrument.setAccuracy(deviceInstrument.getAccuracy());
            oldDeviceInstrument.setMeasureScale(deviceInstrument.getMeasureScale());
            deviceInstrumentRepository.save(oldDeviceInstrument);
        }
        return;
    }
```

对此，我也写了单元测试，单元测试显示成功的更新了数据。但是，正常调用过程中，数据表的数据仍然未改变。我猜想可能是由于`SpringMVC`的缓存机制，数据没有成功更新吧。看来还需要进一步对单元测试与`SpringMVC`的`JPA`加强学习。

# 先删除，再插入
通过前面尝试，我猜想可能是多主键时，`udpate`并没有起作用，也就是说，想使用`update`的方法来直接更新实体信息这个思路是错误的。

解决方法：先删除原有的记录，然后再插入一条新的记录。代码如下：

```java
    @Override
    public void saveOrUpdate(DeviceInstrument deviceInstrument) {
        DeviceInstrument oldDeviceInstrument = deviceInstrumentRepository.findOne(deviceInstrument.getId());
        if (null != oldDeviceInstrument) {
            logger.info("老记录，先删除原记录");
            deviceInstrumentRepository.delete(oldDeviceInstrument);
        }

        deviceInstrumentRepository.save(deviceInstrument);
        return;
    }
```

无论是单元测试还是生产环境，上述代码都得到了期待的结果。

# 总结
在多主键表中，我们需要使用先删除再插入的方法来实现更新操作。