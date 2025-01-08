---
layout:     post
title:     2、检索数据 SELECT + ORDER BY
subtitle:  
date:       2022-07-11
author:     
header-img: 
catalog: true
tags:
    - < database >
typora-root-url: ..
---



# 2、检索数据 SELECT + ORDER BY

> 阅读书籍《SQL必知必会（第4版）》笔记

#### SELECT - 至少两条信息：想选择什么？从什么地方选择？

```sql
SELECT prod_name FROM Products;
SELECT prod_name,prod_id,prod_price FROM Products; # 检索多列用,隔开
```

几个约定俗成：

- 查询结果的顺序不确定 - 随机
- 用  ; 结束 SQL 语句
- SQL 语句不区分大小写、所有空格都会被忽略
- 第一个被检索的行是 第0行
- 注释： --... 、 #... 、  /*...*/
- 检索全部用 * 通配符

#### 关键词

-	DISTINCT：表示数据库只返回不同的值（作用于所有列，而不仅是紧跟其后的列）

```sql
SELECT DISTINCT vend_id FROM Products # DISTINCT 放在列名前面
```

- LIMIT 5 OFFSET 2：返回 第2行起 不超过 5行 的数据

```sql
LIMIT 5 OFFSET 2;
LIMIT 2,5; # 前面的对应 OFFSET，后面的对应 LIMIT
```

- ORDER BY：子句取一个或多个列的名字排序（默认升序排序）
    - 规则① ：必须是 SELECT 语句中的最后一条子句，否则会报错
    - 规则② ：可以使用非检索的列排序

```sql
ORDER BY prod_name;
ORDER BY prod_price,prod_name; # 多个列时，排序分优先级 - 先按价格排、再按名称排
ORDER BY prod_price DESC,prod_name; # DESC，降序排序，只作用于直接位于它前面的列
```

<img src="/../img/assets_2023/925B7437-4E82-4C43-8BFE-A7CD01892C65.png" alt="925B7437-4E82-4C43-8BFE-A7CD01892C65" style="zoom:28%;" />



