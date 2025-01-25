---
layout:     post
title:     sequelize 批量更新的唯一索引
subtitle:  
date:       2024-01-25
author:     
header-img: 
catalog: true
tags:
    - < database >
typora-root-url: ..
---



# sequelize 批量更新的唯一索引

批量更新 daily_data 时，当 时间+序号 重复时，更新数据，而不是再创建一条

步骤1：sequelize 中声明 modal 时声明「唯一索引」

```js
sequelize.define(table, {...}, {
    indexes: [
        {
            unique: true,
            fields: ['date', 'day_sort'] // 为 date 和 sort 字段组合设置唯一索引
        }
    ]
})
```

步骤2：在 mysql 中也进行唯一索引的关联

```mysql
-- 检查是否存在唯一索引
SHOW INDEX FROM `daily` WHERE Column_name IN ('date', 'day_sort');
-- 创建唯一索引
CREATE UNIQUE INDEX idx_date_day_sort ON `daily` (`date`, `day_sort`);
```

步骤3：bulkCreate 配置 updateOnDuplicate 参数

```js
await DailyModal.bulkCreate(body?.data, {
    updateOnDuplicate: [
        'routineTypeId',
        'startTime',
        'endTime',
        'duration',
        'weekday',
        'interval',
    ],
})
```



