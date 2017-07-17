---
title: 在angularJs中进行json数据的格式化
date: 2017-07-17 09:38:49
category: teacherPan
tags: ['angularJs', 'josn', 'format']
---
我们在angularJs的项目中，常常需要对`json`进行显式的查看，以便更清晰的了解到`json`数据内部结构与数值变化。本文将介绍一种非常不错的方法，来协助你查看前台绑定的`json`数据。

<!-- more -->
## 格式化以前
比如：
我们在v层中加入以下代码
```html
{{data}}
```
刷新前台，对应输出格式为:
```json
{"id":1,"name":"区县级器具用户","createTime":null,"updateTime":null,"preWorkflowNode":null,"createUser":null,"districtType":{"id":3,"name":"区\\县","pinyin":"quxian","createUser":null},"workflowType":{"id":1,"name":"适用于器具用户新强检器具审批","pinyin":null,"createTime":null,"updateTime":null,"description":"1.向上级管理部门提出申请；2.管理部门可以转给同区域或子区域的技术机构。3.技术机构可办结可返回给管理部门","createUser":null},"departmentType":{"id":2,"name":"器具用户","pinyin":"qijuyonghu","createTime":0,"updateTime":0,"createUser":null},"containSonDistrict":false}
```
即使是浏览器为我们启用了自动换行，对于想查看`json`结构的我们，也是一件非常困难的事，这时候，我们需要结合`<pre>`与`json`过滤器来共同的定制该显示格式：

## 格式化以后
```html
<pre>{{data | json}}</pre>
```

我们将得到如下格式的`json`数据
```json
{
  "id": 1,
  "name": "区县级器具用户",
  "createTime": null,
  "updateTime": null,
  "preWorkflowNode": null,
  "createUser": null,
  "districtType": {
    "id": 3,
    "name": "区\\县",
    "pinyin": "quxian",
    "createUser": null
  },
  "workflowType": {
    "id": 1,
    "name": "适用于器具用户新强检器具审批",
    "pinyin": null,
    "createTime": null,
    "updateTime": null,
    "description": "1.向上级管理部门提出申请；2.管理部门可以转给同区域或子区域的技术机构。3.技术机构可办结可返回给管理部门",
    "createUser": null
  },
  "departmentType": {
    "id": 2,
    "name": "器具用户",
    "pinyin": "qijuyonghu",
    "createTime": 0,
    "updateTime": 0,
    "createUser": null
  },
  "containSonDistrict": false
}
```

如果你习惯了用4个空格来代替缩进，那么你还可以这样：
```html
<pre>{{data | json : 4}}</pre>
```

刷新页面：
```json
{
    "id": 1,
    "name": "区县级器具用户",
    "createTime": null,
    "updateTime": null,
    "preWorkflowNode": null,
    "createUser": null,
    "districtType": {
        "id": 3,
        "name": "区\\县",
        "pinyin": "quxian",
        "createUser": null
    },
    "workflowType": {
        "id": 1,
        "name": "适用于器具用户新强检器具审批",
        "pinyin": null,
        "createTime": null,
        "updateTime": null,
        "description": "1.向上级管理部门提出申请；2.管理部门可以转给同区域或子区域的技术机构。3.技术机构可办结可返回给管理部门",
        "createUser": null
    },
    "departmentType": {
        "id": 2,
        "name": "器具用户",
        "pinyin": "qijuyonghu",
        "createTime": 0,
        "updateTime": 0,
        "createUser": null
    },
    "containSonDistrict": false
}
```
赶快试试吧。