---
layout:     post
title:    class 组件的 setState 和 函数组件的 useState 的使用差异
subtitle:  
date:       2025-07-27
author:     
header-img: 
catalog: true
tags:
    - < React17+TS4 >
typora-root-url: ..
---



# class 组件的 setState 和 函数组件的 useState 的使用差异

总结：

- `this.setState` 的**第二个参数**可以传递一个回调函数，**在状态更新完成且组件重新渲染后执行**，类似于 `useEffect` 的副作用操作

-	useState 的更新函数**本身不支持回调参数**，可以通过 useEffect 依赖模拟
    -	React 团队认为函数组件的副作用应通过 `useEffect` 显式声明，而非隐藏在 `setState` 回调中。这种设计更符合 React Hooks 的**声明式编程**理念

```js
this.setState({ c: 2 }, () => {
    // c 更新后的后置操作 -> 回调函数会在渲染完成后触发，确保拿到的是最新状态
})

useCount(c => c+1) // 改值可以这么写 -> 若要在值变更渲染后执行回调，这种方式不会等待渲染完成，可能拿不到最新的 DOM 或派生状态
useEffect(() => {
    // 回调
}, [count])
```



