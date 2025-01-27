---
layout:     post
title:     sequelize 处理 Date 问题
subtitle:  
date:       2024-01-27
author:     
header-img: 
catalog: true
tags:
    - < database >
typora-root-url: ..
---



# sequelize 处理 Date 问题

当设置的 date 类型是 DataTypes.Date 时，例如 ’2025-01-27‘

##### 问题

需要查询时，发现 where: { date = ‘2025-1-27’ } 是找不到这个行数据的，sequelize 解析依赖于传递参数的 date 格式

```sql
Executing (default): SELECT `id`, `better`, `front`, `good1`, `good2`, `good3`, `reading`, `sport`, `ted`, `video`, `date` FROM `daily_issue_record` AS `daily_issue_record` WHERE `daily_issue_record`.`date` = '2025-01-25 00:00:00';
```

```sql
Executing (default): SELECT `id`, `better`, `front`, `good1`, `good2`, `good3`, `reading`, `sport`, `ted`, `video`, `date` FROM `daily_issue_record` AS `daily_issue_record` WHERE `daily_issue_record`.`date` = '2025-01-26 15:30:26' LIMIT 1;
```

##### 原因

通过 `GET` 参数传递的 `‘2025-1-27’` 是 STRING 类型，解析为 `'2025-01-25 00:00:00’`

通过 `POST` 参数传递 `dayjs().toDate()`，是 Date 类型，解析为当下时间(东八区减去8小时) `'2025-01-26 15:30:26'`

直接查询 date field的时候，不一定查得到

##### 解决方案： 对 date 进行区间查询

```js
function transDateToWhereOptions(date: Date) {
    // 日期转换 - startOf('date') 东八区自动减去 8小时
    const dateObj = new Date(date)
    const startDate = dayjs(dateObj).startOf('date').add(8, 'hours').toDate();
    const endDate = dayjs(dateObj).add(1, 'day').startOf('date').add(8, 'hours').toDate();

    return {
        where: {
            date: {
                [Op.between]: [startDate, endDate],
            },
        },
    }
}
```

