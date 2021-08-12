---
layout:     post
title:     18-2 Promise和setTimeout
subtitle:  
date:       2021-04-13
author:     
header-img: 
catalog: true
tags:
    - < 浏览器原理 >
typora-root-url: ..
---

# 18-2 Promise和setTimeout

#### 情况1：sleep函数 - 间隔 time 后执行 promise

考点：

`setTimeout` **接收一个函数**，现在传的是 func() - `resolve('hello')` ，需要立即执行拿到返回值 - resolve 被执行，`setTimeout` 失效

```js
// 对比组1
function getData() {
    return new Promise((resolve, reject) => {
        setTimeout(resolve('hello'), 2000); // 马上输出 hello
    })
}
// 对比组2
function getData() {
    return new Promise((resolve, reject) => {
        setTimeout(resolve, 2000, 'hello'); // 2秒后输出 hello
    })
}

getData().then(res => {
    console.log(res)
})
```

sleep 函数实现

```js
const sleep = (time) => {
    return new Promise(resolve => setTimeout(resolve, time))
}
```

#### 情况2： Promise.race

拼多多考题：

实现一个promiseTimeout方法，该方法接收两个参数，第一个参数为promise，第二个参数为number类型。该方法的作用为：

1. 若promise在第二个参数给定的时间内处于pending状态，则返回一个rejected的promise，其reason为new Error("promise time out")
2. 若promise在第二个参数给定的时间内处于非pending状态，则返回值为promise

可以使用全部ES6语法。

```js
let timeout = (time) => new Promise((resolve, reject) => {
    setTimeout(reject, time, new Error('promise time out'));
})
let promisesetTimeout = (promise, time) => {
    return Promise.race([promise, timeout(time)])
}
let p = (time) => new Promise((resolve) => {
    setTimeout(resolve, time, '俊哲')
})
console.log(promisesetTimeout(p(5000), 1000));
```

几个要点：

1、 timeout 作用是 间隔 time 后抛出 error 的 Promise

2、Promise.race 是只要有一个参数的 promise 的状态改变，整个返回的 Promise 状态就会跟着改变

3、p 是自己造的延迟 time 后会改变状态为 `resolve` 的 Promise
