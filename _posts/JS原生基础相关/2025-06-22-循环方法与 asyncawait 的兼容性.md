---
layout:     post
title:      循环方法与 async/await 的兼容性
subtitle:  
date:       2025-06-22
author:     
header-img: 
catalog: true
tags:
    - < JS原生基础相关 >
typora-root-url: ..
---



## 循环方法与 async/await 的兼容性

场景：forEach 遍历过程中 await 接口返回失效

- 总结：

    1、不能和 async/await 一起使用的循环方法：**forEach、map、filter、reduce**
    2、能够和 async/await 一起使用的循环方法：**for循环、for-of、for-in**



#### 一、好用的 异步并行执行 的写法

##### 并行：for-of/map 中同时启动所有异步任务 + Promise.all

```js
async function foo(things){
    const results = [];
    for(const thing of things) {
        results.push(bar(thing)) // 立即执行并返回 Promise，所有异步任务同时启动
    }
    return baz(await Promise.all(results)) // 等待所有任务完成，并使用 baz 处理结果
}
// 【更优】整个函数都可以更简洁
async function foo(things){
    const results = await Promise.allSettled(things.map(it => bar(it)))
    return baz(results)
}
```

##### 串行：在 for-of 中 await

```js
// 【较差写法】- 串行
async function foo(things){
    const results = [];
    for(const thing of things) {
        results.push(await bar(thing)) // 每个任务串行执行，效率低
    }
    return baz(results)
}
```





#### 二、为什么在一些循环中无法使用

##### 1、forEach

forEach 不等待异步回调，会立即 **同步遍历所有元素**，不会等待回调函数中的异步操作完成

- 即使回调函数标记为 async，forEach 也不会等待，会继续下一次迭代

```js
// 不会按预期工作
array.forEach(async (it) => {
  await someAsyncOperation(it); // 不会被等待
});
```

##### 2、map

map 会返回包含 promise 的数组，但本身**不会等待任何 promise 解决**

```js
// 不会按预期工作
const results = await array.map(async (item) => {
  return await someAsyncOperation(item); // 返回的是 Promise ，而不是解决后的值数组
});
```

##### 3、filter

filter 同步执行，无法等待异步条件判断

- async 函数返回 Promise，而 Promise 是 truthy 值，所以所有元素通常都会被保留

```js
// 不会按预期工作
const filtered = array.filter(async (item) => {
  return await someAsyncCheck(item); // 总是返回 Promise(truthy)
});
```

##### 4、reduce

reduce 累加器无法正确处理异步操作，**会同步执行所有迭代**

- 会导致累加器接收 Promise 而不是解决后的值

```js
// 不会按预期工作
const result = await array.reduce(async (acc, item) => {
  const resolvedAcc = await acc;
  return resolvedAcc + await someAsyncOperation(item);
}, Promise.resolve(0)); // 虽然能工作，但代码复杂且容易出错
```

