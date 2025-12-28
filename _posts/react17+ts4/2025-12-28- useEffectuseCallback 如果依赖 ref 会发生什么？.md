---
layout:     post
title:    useEffect/useCallback 如果依赖 ref 会发生什么？
subtitle:  
date:       2025-12-28
author:     
header-img: 
catalog: true
tags:
    - < React17+TS4 >
typora-root-url: ..
---



# useEffect/useCallback 如果依赖 ref 会发生什么？

总结：

1、ref.current 的赋值方式如果是 `  ref.current = { value: count }` 覆盖式的，如果作为 useEffect/useCallback 的依赖，会触发无限循环

2、核心原则：ref 用于存储 **可变但不触发渲染的值**，如果要响应变化，应该用 state

> 关键知识点纠偏

useEffect/useCallback 的依赖项是 <span style="color: red">浅比较</span>（`Object.is()` 值比较+**引用比较**），只比较引用地址



### 场景1：ref.current 作为 useEffect 的依赖对象

不行，无法重新渲染

因为 `dataRef.current` 返回的 ref 对象在整个生命周期中的 **引用是不变化** 的，current 的变化不会触发重新渲染

### 场景2：ref.current 作为 useCallback 的依赖对象

不行，可能会无限循环（不是所有 ref.current 都会触发无限循环的，主要是看 ref 更新的方式）

- 1、在渲染期间 更新了 ref.current 【无限循环】

    在渲染期间 更新了 ref.current -> 导致 useCallback 依赖的函数也更新 -> 如果 useEffect 依赖这个函数，那么就会触发useEffect 里面的副作用 -> 副作用如果更新 state 就会触发重新渲染 -> 循此往复

    ```js
    ref.current = count + 1; // 每次渲染都修改 ref.current
    ```

- 2、ref.current 是对象，属性式更新【无法重新渲染】

    ref 和 ref.current 此时都是同一个引用地址，无法触发重新渲染

    ```js
    ref.current.count++
    ```

- 3、ref.current 是对象，赋值式更新【无限循环】

    永远都是新对象，导致一直触发函数的重新创建

    ```js
    ref.current = { value: count }; // ❌ 创建新对象！
    ```

    





