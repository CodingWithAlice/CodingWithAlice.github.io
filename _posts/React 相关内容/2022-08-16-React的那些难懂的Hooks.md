---
layout:     post
title:      React那些难懂的Hooks
subtitle:  
date:       2022-08-16
author:     
header-img: 
catalog: true
tags:
    - < React >
typora-root-url: ..
---

# React那些难懂的Hooks

## 背景

【开骂】垃圾 React 官网，hooks 描述只有晦涩难懂的几个词，用起来一踩一个坑，贼准

- 推荐学习链接

[React 开发者Dan的文章](https://overreacted.io/zh-hans/a-complete-guide-to-useeffect/)（推荐，React 开发者）、[React Hooks 最佳实践](https://juejin.cn/post/6844904165500518414)（云音乐的最佳实践）



## 重点摘要

推荐阅读：

- [如何在useEffect里做数据请求](https://www.robinwieruch.de/react-hooks-fetch-data/) // todo

关键点：

- **Effects 拿到的，总是定义它那次渲染中的 props 和 state**
- 如果想要读取最新值，最简单的方法是使用 refs，详细： [这篇文章的最后一部分](https://overreacted.io/how-are-function-components-different-from-classes/) // todo



## 详解

```react
function Counter() {
  const [count, setCount] = useState(0);
  
  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + count);
    }, 3000);
  }
  
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
      <button onClick={handleAlertClick}>
        Show alert
      </button>
    </div>
  );
}
```

### 1、每一次渲染都有它自己的 Props and State

当`setCount`的时候，React会**带着一个不同的`count`值再次调用组件**。然后，React会更新DOM以保持和渲染输出一致；

任意一次渲染中的`count`常量都不会随着时间改变。**渲染输出会变是因为我们的组件被一次次调用**。



### 2、每一次渲染都有它自己的事件处理函数

组件的函数每次渲染都会被调用，但是每一次调用中count值都是常量，并且它被赋予了当前渲染中的状态值；

每一次渲染都有一个**“新版本”的`handleAlertClick；`**

每一个版本的`handleAlertClick`“记住” 了**它自己的 `count`；**

事件处理函数**“属于”某一次特定的渲染**，当你点击的时候，它会使用那次渲染中counter的状态值。



### 3、每次渲染都有它自己的Effects

并不是count的值在“不变”的effect中发生了改变，而是**effect 函数本身在每一次渲染中都不相同**；

每一个effect版本“看到”的`count`值（props 和 state ）都来自于它属于的那次渲染；

React会记住你提供的effect函数，并且会在每次更改作用于DOM并让浏览器绘制屏幕后去调用它 -- **概念上，你可以想象effects是渲染结果的一部分**。



### 总结：

**在组件内什么时候去读取props或者state是无关紧要的**。因为它们不会改变。在单次渲染的范围内，props和state始终保持不变。

<img src="/../img/assets_2019/:Users:haoling:Library:Application Support:typora-user-images:image-20220822213608238.png" alt="image-20220822213608238" style="zoom:80%;" />









