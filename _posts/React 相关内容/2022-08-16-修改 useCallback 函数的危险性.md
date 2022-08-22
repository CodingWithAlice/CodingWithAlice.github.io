---
layout:     post
title:     修改 useCallback 函数的危险性
subtitle:  
date:       2022-08-16
author:     
header-img: 
catalog: true
tags:
    - < React >
typora-root-url: ..
---


# 修改 useCallback 函数的危险性

useCallback hooks 生成的函数，在依赖不变的情况下，这个函数是不会变化的；

##### 功能

新增一个 lastStatus 存储 上次 status 的状态

##### 问题

当前函数使用了 useCallback ，只在 appId 变更（初始化时）生成函数，后续再访问当前函数，变量 status 一直为 ''，无法读取到上一次接口的值

```javascript
 const getStatus = useCallback(() => {
    return Api.check({ appId, envType })
      .then((res) => {
        setLastStatus(status); // 此次新增的 lastStatus 变量，存储上一次查询接口时的 status 的状态
        setStatus(res.status);
      })
  }, [appId]);
```

##### 解决办法

给函数新增依赖 status

- 风险：由于新增 status 依赖，status 变化会导致当前函数 getStatus 更新，需要确认当前使用 getStatus 函数的地方，有没有依赖 getStatus，或者 **依赖 getStatus 的功能影响可控**

```react
 const getStatus = useCallback(() => {
    return Api.check({ appId, envType })
      .then((res) => {
        setLastStatus(status); // 此次新增的 lastStatus 变量，存储上一次查询接口时的 status 的状态
        setStatus(res.status);
      })
  }, [appId, status]); // 此次新增 status 变量作为依赖
```

