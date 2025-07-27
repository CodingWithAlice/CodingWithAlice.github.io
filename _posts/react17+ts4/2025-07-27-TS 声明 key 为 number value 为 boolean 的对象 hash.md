---
layout:     post
title:    TS 声明 key 为 number value 为 boolean 的对象 hash
subtitle:  
date:       2025-07-27
author:     
header-img: 
catalog: true
tags:
    - < React17+TS4 >
typora-root-url: ..
---



# TS 声明 key 为 number value 为 boolean 的对象 hash

总结：

> 方案一、二 底层逻辑完全一致

- 方案1：索引签名

```js
const hash: { [key: number]: boolean } = {}
```

- 方案2：内置工具类 Record
    - Record<number, boolean> 是 TS 的内置工具类 -> 底层依然是索引签名
    - 表示一个对象类型，键为 number，值为 boolean

```js
const hash: Record<number, boolean> = {}
```

