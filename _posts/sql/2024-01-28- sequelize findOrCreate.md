---
layout:     post
title:     sequelize findOrCreate
subtitle:  
date:       2024-01-28
author:     
header-img: 
catalog: true
tags:
    - < database >
typora-root-url: ..
---



# sequelize findOrCreate

##### 问题

使用 findOrCreate 后能 创建新数据，不能更新

##### 原因

函数的作用是：在数据库中「查找一条记录」，如果找不到，则「创建一条新记录」

​	-–> 只能查找和创建，如果需要更新，需要将数据更新到数据库

使用方式

```js
const [instance, created] = Model.findOrCreate({ where, defaults })
options:{ where, defaults } 配置对象 - 查询条件和创建数据
instance - 找到/创建的记录实例
created - 布尔值，表示是否创建了新纪录
```

##### 解决方案

```js
const [issue, created] = await WeekModal.findOrCreate({
    where: { serialNumber: data.serialNumber },
    defaults: data,
})
if (!created) {
    issue.set(data);
    // 如果已经存在，更新描述
    await issue.save()
}
return NextResponse.json({ success: true, message: created ? '观察成功' : '更新成功' })
```

