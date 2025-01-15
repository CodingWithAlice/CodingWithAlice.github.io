---
layout:     post
title:     sequelize遇到mysql2安装问题
subtitle:  
date:       2024-01-15
author:     
header-img: 
catalog: true
tags:
    - < database >
typora-root-url: ..
---



# sequelize遇到mysql2安装问题

```json
// 版本信息
{
    node: 'v18.20.5',
    "mysql2": "^2.3.3",
    "sequelize": "^6.37.5",
}
```

代码正确使用方式

```js
import { Sequelize } from 'sequelize';
import mysql2 from 'mysql2';

export const sequelize = new Sequelize('Daily', 'root', 'localhost', {
  host: 'localhost',
  dialect: 'mysql',
  dialectModule: mysql2, // 解决
})
```

遇到的问题：`Error: Please install mysql2 package manually`

原因：Sequelize 实例化的时候，使用的方言是 `dialect: 'mysql'`，Sequelize 对于 MySQL 使用的基础连接器库是 [mysql2](https://www.npmjs.com/package/mysql2) npm 软件包(1.5.2 或更高版本)

困难：

- 安装 mysql2 高版本可能会遇到兼容问题
- 即使安装后，可能也会出现上述提示，添加 `dialectModule: mysql2` 可解决



