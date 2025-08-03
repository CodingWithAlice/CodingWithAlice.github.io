---
layout:     post
title:      lodash 的 isEqual和 cloneDeep
subtitle:  
date:       2025-08-02
author:     
header-img: 
catalog: true
tags:
    - < JS原生基础相关 >
typora-root-url: ..
---



## lodash 的 isEqual和 cloneDeep

- 总结：

    1、isEqual 深度比较
    
    - 函数的比较规则：【浅比较（只比较代码，不比较闭包）】先判断是不是 同一个引用 -> 如果不是，判断`toString()` 转换后代码是否一致 -> **不会深入比较闭包 + this** (有些变量值运行时确定，无法静态解析)
    
    2、cloneDeep 深度拷贝



### 一、isEqual 深度比较

- 基本类型 number string boolean null undefined Symbol -> 直接 === 处理（NaN 特殊处理）
- 引用类型
    - 对象类型：
        - 先比较引用地址（`===`），如果相同则直接返回 `true`
        - 比较构造函数（`Object`、`Array`、`Date`、`RegExp` 等），如果不一致则返回 `false`
        - 递归比较所有属性和子属性
    - 其他类型
        - **Date**：比较 `.getTime()`
        - **RegExp**：比较 `.source` 和 `.flags`
        - **Map/Set**：递归比较键和值
        - **Buffer/TypedArray**：逐个比较字节

```JS
function isEqual(a, b) {
  // 基本类型比较
  if (a === b) return true;

  // 处理 NaN
  if (Number.isNaN(a) && Number.isNaN(b)) return true;

  // 类型不同直接返回 false
  if (typeof a !== 'object' || typeof b !== 'object' || a === null || b === null) {
    return false;
  }

  // 处理 Date
  if (a instanceof Date && b instanceof Date) {
    return a.getTime() === b.getTime();
  }

  // 处理 RegExp
  if (a instanceof RegExp && b instanceof RegExp) {
    return a.source === b.source && a.flags === b.flags;
  }

  // 比较对象/数组的键
  const keysA = Object.keys(a);
  const keysB = Object.keys(b);

  if (keysA.length !== keysB.length) return false;

  for (const key of keysA) {
    if (!keysB.includes(key) || !isEqual(a[key], b[key])) {
      return false;
    }
  }

  return true;
}
```



### 二、cloneDeep 深度克隆

1. **基本类型**：直接返回
2. **对象类型**：
    - 处理循环引用（使用 `WeakMap` 缓存已克隆的对象）
    - 根据不同类型进行克隆：
        - **Array** → 递归克隆每一项
        - **Object** → 递归克隆所有属性。
        - **Date** → `new Date(value.getTime())`
        - **RegExp** → `new RegExp(value.source, value.flags)`
        - **Map/Set** → 遍历并递归克隆每一项
3. **特殊类型**：
    - **Buffer/TypedArray**：使用 `Buffer.from(value)` 或 `new TypedArray(value)`

```js
function cloneDeep(value, cache = new WeakMap()) {
  // 基本类型直接返回
  if (typeof value !== 'object' || value === null) {
    return value;
  }

  // 处理循环引用
  if (cache.has(value)) {
    return cache.get(value);
  }

  // 处理 Date
  if (value instanceof Date) {
    return new Date(value.getTime());
  }

  // 处理 RegExp
  if (value instanceof RegExp) {
    return new RegExp(value.source, value.flags);
  }

  // 处理 Array
  if (Array.isArray(value)) {
    const clonedArray = [];
    cache.set(value, clonedArray);
    for (const item of value) {
      clonedArray.push(cloneDeep(item, cache));
    }
    return clonedArray;
  }

  // 处理普通对象
  const clonedObj = {};
  cache.set(value, clonedObj);
  for (const key in value) {
    if (value.hasOwnProperty(key)) {
      clonedObj[key] = cloneDeep(value[key], cache);
    }
  }

  return clonedObj;
}
```

