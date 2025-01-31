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

补充

详细看下 [sequelize.define 可以接收的第三个参数 - 常用配置属性](https://sequelize.org/docs/v6/core-concepts/model-basics/#model-definition)

```js
const User = sequelize.define('User', {
    // 字段定义
}, {
    // 1、tableName - 指定数据库中的表名
    tableName: 'users', // 指定表名为 'users'
    // 2、timestamps - 是否自动添加 createdAt 和 updatedAt 字段，默认为 true
    timestamps: false, // 禁用时间戳字段
    // 3、paranoid - 启用软删除（即删除时不会真正删除数据，而是设置 deletedAt 字段），默认为 false
    paranoid: true, // 启用软删除
    // 4、underscored - 将字段名自动转换为下划线命名，默认为 false
    underscored: true, // 将 createdAt 映射为 created_at
    // 5、freezeTableName - 禁止 Sequelize 自动将表名转换为复数形式，默认为 false
    freezeTableName: true, // 表名保持为 'User'，而不是 'Users'
    // 6、indexes - 定义表的索引
    indexes: [
        {
            unique: true,
            fields: ['email'] // 在 email 字段上创建唯一索引
        }
    ]
});
```

