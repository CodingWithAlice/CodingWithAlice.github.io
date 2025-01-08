---
layout:     post
title:     6、分组数据 GROUP BY + HAVING
subtitle:  
date:       2022-07-11
author:     
header-img: 
catalog: true
tags:
    - < database >
typora-root-url: ..
---



# 6、分组数据 GROUP BY + HAVING

> 阅读书籍《SQL必知必会（第4版）》笔记

使用分组可以将数据分为多个逻辑组，对每个组进行聚集计算

#### 1、GROUP BY - 创建分组

- GROUP BY 可以包含任意数目的列：因而可以对分组进行嵌套，进行更细致的分组
    - 若嵌套，数据在最后指定的分组上进行汇总，所有的列都一起计算，不能从个别列取回数据
- GROUP BY 中列出的每一列都必须是检索列或者有效表达式（不能是聚集函数，不能使用 SELECT 中定义的别名）
    - 一般不允许 GROUP BY 列带有长度可变的数据类型（文本、备注型字段）
    - 除聚集函数外，SELECT 中的每一列，都必须在 GROUP BY 中给出
    - 如果分组列中含有 NULL 值的行，NULL 将作为一个分组返回
- GROUP BY 子句必须在 WHERE 子句之后，ORDER BY 子句之前

```sql
SELECT vend_id, COUNT(*) AS num_prods
FROM Products
GROUP BY vend_id;
-- 得到结果
vend_id num_prods
BRS1    3
DLL01   4
FNG1    2
```

<img src="/../img/assets_2023/C0C02F17-3F0C-45BA-9F7A-34AE0BBF83AA.png" alt="C0C02F17-3F0C-45BA-9F7A-34AE0BBF83AA" style="zoom:30%;" />

#### 2、HAVING 过滤分组

> 建议配套 GROUP BY 使用

HAVING 和 WHERE很像，只不过 WHERE 用来 **过滤行**，HAVING 用来**过滤分组**

WHERE 在数据 **分组前进行过滤**，HAVING 在数据 **分组后进行过滤**

```sql
SELECT vend_id, COUNT(*) AS total_prods
FROM Products
WHERE prod_price > 5 # 表示商品价格大于5
GROUP BY vend_id
HAVING COUNT(*) >= 3; # 表示过滤3个产品及以上的供应商
```

