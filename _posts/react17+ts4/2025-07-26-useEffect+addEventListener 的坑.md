---
layout:     post
title:    useEffect+addEventListener 的坑
subtitle:  
date:       2025-07-26
author:     
header-img: 
catalog: true
tags:
    - < React17+TS4 >
typora-root-url: ..
---



# useEffect+addEventListener 的坑

总结：

- 即使 React 组件重新渲染，事件监听绑定的依旧是 当时的函数引用，捕获的是当时的作用域 和变量



##### 背景问题

​	addEventListener 事件监听器绑定的回调函数，一直是旧的

##### 原因

​	addEventListener 事件监听器 **绑定时** 捕获的是 **当时的** props 和 state（当时的作用域和变量，旧的） -> 后续不会自动变化

- 知识点1：addEventListener 绑定的是 **当时的函数引用**，后续 props 变化不会自动影响已经绑定的事件处理函数
- 知识点2：React 的函数组件每次渲染都会生成新的函数，但 **事件监听不会自动更新为新函数**，除非手动解绑再绑定

##### 方案

​	必须在 useEffect 依赖里面，解绑 旧的，绑定 新的 -> 这样事件处理函数才会用到最新的 props

```js
useEffect(() => {
    window.addEventListener('scroll', handleScroll);
    return () => {
      window.removeEventListener('scroll', handleScroll); // 清理旧监听
    };
  }, [handleScroll]); // 依赖 handleScroll（其变化时会重新绑定）
```



