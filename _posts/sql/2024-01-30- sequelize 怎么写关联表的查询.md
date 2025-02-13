---
layout:     post
title:     sequelize 怎么写关联表的查询
subtitle:  
date:       2025-01-28
author:     
header-img: 
catalog: true
tags:
    - < database >
typora-root-url: ..
---



# sequelize 怎么写关联表的查询

##### 问题

查询每日数据表，想要组合「按照日期」查询

```js
const dailyTimeRecord = await TimeModal.findAll(options)
const dailyIssueRecord = await IssueModal.findAll(options)
```

使用 on 关联查询 - 已关联的两张表，不需要再使用 on - 会默认添加 - 更方便管理

```js
const weekData = await IssueModal.findAll({
    include: [{model: TimeModal}]
})
```

提示 Error [SequelizeEagerLoadingError]: daily_issue_record is not associated to daily_time_record!

##### 原因

需要关联后，才能够关联查询

##### 解决方案

```js
// 关联关系
IssueModal.hasMany(TimeModal, {
	foreignKey: 'date', // 多的键
	sourceKey: 'date',
})
TimeModal.belongsTo(IssueModal, {
	foreignKey: 'date', // 多的键
	targetKey: 'date',
})
```

