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

如果是更新，需要 modal 保存入数据库

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

