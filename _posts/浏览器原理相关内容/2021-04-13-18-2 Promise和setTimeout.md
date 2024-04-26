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

[可执行代码链接](https://github.com/CodingWithAlice/init-my/blob/master/src/assets/%E6%89%8B%E6%92%95/promiseSettimeout.html)

#### 第一步：sleep函数的实现（借由 setTimeout）

##### 先深入理解`setTimeout` 函数

<img src="/../img/assets_2023/image-20240426165715508.png" alt="image-20240426165715508" style="zoom:25%;" />

```js
// MDN 对 setTimeout 的描述
setTimeout(functionRef, delay, param1, param2, /* … ,*/ paramN)
```

如上图，

- 示例1：第一个参数是 `resolve('hello')`时 ，其实是传入了 `resolve('hello')`的结果，会立即执行拿到返回值，`setTimeout` 到期后返回一个一个 `fullfilled` 状态的 `Promise`

- 示例2：第一个参数是 `resolve`，在`setTimeout` 到期后将 ‘hello’ 作为参数执行 `resolve`

##### sleep 函数实现

由上述例子可以得到，可以明显参考示例2，实现 sleep 指定时长后执行指定操作

```js
const sleep = (time) => {
    return new Promise(resolve => setTimeout(resolve, time))
}
sleep(1000).then(() => {
    // 这里写你的操作
    console.log('鸡丢了');
})
```

##### 再深入一点 - sleepAsync 函数实现

```js
async function sleepAsync() {
    console.log('1')
    let res = await sleep(1000)
    console.log('2')
    return res
}
sleepAsync()
```



#### 第二步：实现 promisesetTimeout（拼多多面试题）

题目：实现一个promisesetTimeout方法，该方法接收两个参数：第一个参数为promise，第二个参数为number类型

结果需满足：

1）若`Promise`在第二个参数给定的时间内处于`pending`状态，则返回一个`rejected`的`Promise`，其reason为`new Error('promise time out')`

2）若`Promise`在第二个参数给定的时间内处于非`pengding`状态，则返回该`Promise`

##### 题目拆解：

1、 超时函数：间隔 time 后抛出 error 的 Promise（即上文提及的 sleep 函数）

2、Promise.race（核心） ：只要有一个参数的 promise 的状态改变，整个返回的 Promise 状态就会跟着改变

```js
// 超时函数 - 返回一个自定义睡眠时长的，用于抛出错误的 Promise
let timeout = (time) => new Promise((resolve, reject) => {
    setTimeout(reject, time, new Error('promise time out'));
})
// 核心：通过 Promise.race 改变返回的 Promise 结果的状态
let promiseSetTimeout = (promise, time) => {
    return Promise.race([promise, timeout(time)])
}

// 使用示例：
// 构建一个 promise 作为第一个参数 - sleep 函数实现
let p = (time) => new Promise((resolve) => {
    setTimeout(resolve, time, 'rps')
})
console.log(promiseSetTimeout(p(5000), 1000));
```

