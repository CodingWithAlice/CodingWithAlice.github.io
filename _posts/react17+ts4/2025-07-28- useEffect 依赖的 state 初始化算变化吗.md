---
layout:     post
title:    useEffect 依赖的 state 初始化算变化吗
subtitle:  
date:       2025-07-28
author:     
header-img: 
catalog: true
tags:
    - < React17+TS4 >
typora-root-url: ..
---



# useEffect 依赖的 state 初始化算变化吗

总结：state 初始化是否算“变化”？

- 不算。useState(0) 的初始赋值是同步的，useEffect 只是在**挂载后检查依赖时**发现 count 存在（值为 0） -> 但原本 挂载后， useEffect 的副作用会被执行一次，因此只执行一次



1、依赖项为 **空数组**

- 组件挂载（初次渲染）后，立即执行一次（类似于 componentDidMount）

```js
useEffect(() => {
    callbakc() // 只执行一次
}, []) 
```

2、依赖项为父级传递的 props

- 组件挂载后，立即执行一次
- props.status 变化后，触发执行一次
    - 从 `undefined` 到初始值的 **首次赋值也会触发执行**

```js
useEffect(() => {
    callbakc()
}, [props.status]) // 依赖 props.status
```

3、依赖项为 当前组件的 state

- 组件挂载后，立即执行一次
- count 改变，触发执行一次
    - `state` 的 **初始赋值不算作“变化”**，但会触发首次副作用（因为 `useEffect` 在挂载后总会运行一次）

```js
const [count, setCount] = useState(0);
useEffect(() => {
    callbakc()
}, [count])
```

