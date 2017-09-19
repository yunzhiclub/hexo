---
title: 如何给某个java.sql.Date加上固定的天数
date: 2017-08-23 11:34:39
tags: ['java', 'sql.Date']
category: teacherPan
---
在计量项目中，碰到了如下问题：
1. 计量器具设置了上次的检定日期。
2. 计量器具设置了检定周期（天）

我们想得到：计量器具下次应该去进行检定的时间。

算法：上次的检定日期 + 检定周期 = 下次应该检定的日期。
<!-- more  -->
<hr />

那么，在JAVA中，是如何做到在一个日期类型上加入固定的天数呢？
这里我们需要用到借助`Calendar`类中的`add`方法来实现。
```java
class Test {
      /**
       * java.sql.Date加上固定的天数
       * @param    {java.sql.Date}                 java.sql.Data date 原始日期
       * @param    {int}                 int           days 加上的天数
       * @author 梦云智 http://www.mengyunzhi.com
       * @DateTime 2017-09-19T14:56:25+0800
       */
      public java.sql.Date addDate(java.sql.Data date, int days) {
            java.util.Calendar calendar = java.util.Calendar.getInstance();   // 实例化
            calendar.setTime(date);                                           // 设置日期
            calendar.add(calendar.DATE, days);                                // 与 天数 相加
            return new java.sql.Date(calendar.getTimeInMillis());             // 使用calendar.getTimeInMillis()实例化java.sql.Date
      }
}
            
```



