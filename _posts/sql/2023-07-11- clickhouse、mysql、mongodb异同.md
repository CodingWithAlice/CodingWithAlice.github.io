---
layout:     post
title:     clickhouse、mysql、mongodb异同
subtitle:  
date:       2023-07-11
author:     
header-img: 
catalog: true
tags:
    - < database >
typora-root-url: ..
---



# clickhouse、mysql、mongodb异同

- 总结

|                | MySQL-关系型                         | MongoDB-文档型                                 | ClickHouse-列式数据库                                |
| -------------- | ------------------------------------ | ---------------------------------------------- | ---------------------------------------------------- |
| 性能           | 较好                                 | 读的性能好，写的较弱                           | 读写性能高                                           |
| 适用场景       | 事务处理（增删改查），适合中小型应用 | 半结构化(XML、HTML、JSON等)/非结构化的数据存储 | 大规模数据的分析和查询：高吞吐、低延迟的场景（日志） |
| 数据一致性问题 | 高并发时可能出现数据一致性问题       | 在分布式场景下可能出现数据一致性问题           | 不支持事务，支持复制和分布查询                       |



ClickHouse、MySQL 和 MongoDB 是三种不同类型的数据库管理系统，它们的区别和优缺点如下：

数据模型：

- ClickHouse 是一种 **列式数据库**，适合于高吞吐量、低延迟的分析场景；
- MySQL 是一种 **关系型数据库**，适合于事务处理和 OLTP 场景；
- MongoDB 是一种 **文档型数据库**，适合于 **半结构化和非结构化数据的存储**。

性能：

- ClickHouse 的读写性能非常高，适合于 **大规模数据** 的分析和查询；
- MySQL 的性能较好，适合于 **中小型应用**；
- MongoDB 的 **读性能较好**，写性能较弱。

数据一致性：

- ClickHouse 不支持事务，但支持复制和分布式查询；
- MySQL 支持事务，但在高并发场景下可能会出现数据一致性问题；
- MongoDB 支持 ACID 事务，但在分布式场景下可能会出现数据一致性问题。

索引（单独存储的、用于查询的数据结构）：

- ClickHouse 的索引支持范围查询和排序，适合于大规模数据的分析和查询；
- MySQL 的索引支持多种类型，适合于各种场景；
- MongoDB 的索引支持嵌套和复合类型，适合于半结构化和非结构化数据的查询。

扩展性：

- ClickHouse 和 MongoDB 都支持分布式架构和水平扩展，适合于大规模数据的处理和分析；
- MySQL 支持主从复制和分区表，但在大规模数据场景下可能会出现性能瓶颈。

总体来说，ClickHouse 适合于大规模数据的分析和查询，MySQL 适合于事务处理和 OLTP 场景（On - Line Transaction Processing 联机事务处理），MongoDB 适合于半结构化和非结构化数据的存储和查询



