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

总结：

- 部署上线后，由于线上服务器的时间和本地可能存在差异，使用 dayjs.utc 解决

```js
import dayjs from 'dayjs'
import utc from 'dayjs/plugin/utc'
dayjs.extend(utc);

function transOneDateToWhereOptions(date: string) {
    const dateObj = new Date(date)
    const startDate = dayjs.utc(dateObj).startOf('day').toDate()
    const endDate = dayjs.utc(dateObj).endOf('day').toDate()
    const where2 = {
        date: {
            [Op.between]: [startDate, endDate],
        },
    }
    return where2
}
```

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

##### 解决方案：

##### 方法一【推荐】：对 date 进行区间查询

```js
function transDateToWhereOptions(date: string) {
	// 日期转换 - startOf('date') 东八区自动减去 8小时 -> 使用 utc 解决
	const dateObj = new Date(date)
	const startDate = dayjs(dateObj).startOf('date').add(8, 'hours').toDate()
	const endDate = dayjs(dateObj).endOf('date').add(8, 'hours').toDate()
	// 方法1：使用 Op.between 来匹配日期范围
	const where2 = {
		date: {
			[Op.between]: [startDate, endDate],
		},
	}

	return {
		where: where2,
	}
}
```

推荐原因： 性能考虑 - 在有索引的情况下，对于大数据量的查询，**`Op.between` 通常性能最高**；`DATE` 函数次之；`DATE_FORMAT` 性能相对较差

- `DATE_FORMAT`：数据库通常「无法使用索引来加速查询」。

    因为函数会对索引列进行计算，使得数据库不能直接根据索引定位到符合条件的记录，**需要扫描整个表或者索引范围**，所以在大数据量的情况下，性能可能较差

- `DATE`：DATE 函数相对 DATE_FORMAT 对索引的影响较小。如果 date 上有索引，数据库可以在一定程度上利用索引。因为 DATE 函数只是提取日期部分，操作相对简单，不会像 DATE_FORMAT 那样完全破坏索引的有序性，性能一般比 DATE_FORMAT 要好。

- `[Op.between]`：如果 date_column 上有索引，**区间查询可以有效地利用索引**。数据库可以根据索引快速定位到区间的起始和结束位置，然后扫描索引范围内的记录，避免全表扫描。所以在合适的索引支持下，Op.between 的性能通常是比较高的。

##### 方法二：将数据转换成 DATE 再匹配

```js
// 方法1：使用 DATE_FORMAT/DATE 来转换成 YYYY-MM-DD 匹配是否和 参数相等
const where1 = Sequelize.where(
    Sequelize.fn('DATE_FORMAT', Sequelize.col('date'), '%Y-%m-%d'),
    '=',
    date
)
const where3 = Sequelize.where(
    Sequelize.fn('DATE', Sequelize.col('date')),
    '=',
    date
)
```

