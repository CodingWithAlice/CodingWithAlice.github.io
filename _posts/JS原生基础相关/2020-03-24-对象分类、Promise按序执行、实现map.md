---
layout:     post
title:     对象分类、Promise按序执行、实现map
subtitle:  
date:       2020-07-07
author:     
header-img: 
catalog: true
tags:
    - < JS原生基础相关 >
typora-root-url: ..
---

# 对象分类、Promise按序执行、实现map

![image-20241023104802308](/../img/assets_2023/image-20241023104802308.png)

```js
// 两个参数：归并函数，归并起点初始值
arr.reduce(callback(), initialValue);
callback(prev, cur[, curIndex[, array]]);
```

注意：第一次执行时， `pre` 和 `cur` 的取值有两种情况：

- 如果提供了`initialValue`，`pre` **取值为 `initialValue`**，`cur` **取数组中的第一个值**

- 如果没有提供 `initialValue`，那么 `pre` **取数组中的第一个值**，`cur`**取数组中的第二个值**



#### 补充：`reduce` 能否配合 `then` / `async/await`？

- 结论：`reduce` 本身是**同步遍历**没错，但它完全可以用来**串行**执行 Promise/async 任务；关键在于把 **累加器设计成 Promise**，并且在每次迭代中**正确地 return / await 上一次的结果**，从而形成「Promise 链」。
- 常见误区：不是 “`reduce` 不能配合 `then` / `async/await`”，而是「把异步值当成同步值用」或「没有把 Promise 链 return 出去」，导致你看到 `splice/push` 之类的副作用像是“没生效”。
- 联网核对：这种 “`reduce` + `pre.then(cur)` 串行执行任务” 的写法是社区长期通用模式；`async/await` 版的关键点同样是把累加器当作 Promise 来 `await`（可参考 Stack Overflow / CSS-Tricks 等对该模式的解释）。

##### 反例1：`reduce` + `async/await`（没有把累加器当 Promise 正确串起来）

- 现象：`reduce` 中 `splice` / `push` 表现异常
- 本质原因：回调是 `async` 时返回的是 Promise；如果你**没有围绕“累加器 Promise”来组织时序**（例如没有用 `prePromise` 串起来），就会在多个异步回调交错时修改同一个 `initList`，产生竞态（race condition）

```js
function execute(questList) { // 执行所有请求，返回所有响应
    questList = transList(questList); // 闭包 -封装 index 和所有请求
    let initList = questList.splice(0, 6); // 初始化6个
    questList = questList.concat(Array(6)) // 补齐最后的空，方便执行完 Promise.race
    return questList.reduce(async (pre, cur) => { ❌
        const { key, value } = await Promise.race(initList.map(it => it.promise));
        const resolvedIndex = initList.findIndex(it => +it.index === +key);
        initList.splice(resolvedIndex, 1); // 移除已返回的 promise
        pre[key] = value; // 存储响应值
        !!cur && initList.push(cur); // 添加新的 promise
        return pre;
    }, [])
}
```
##### 反例2：`reduce` + `then`（没有把链 return 出去 / 累加器类型不一致）

- 现象：你以为“下一个迭代会拿到上一个 `then` 更新后的值”，但实际上 `reduce` 只认**本次回调的返回值**
- 本质原因：如果没有 `return pre.then(...)`（或返回一个新的 Promise 作为累加器），`reduce` 的累加器就不会等待异步完成；后续迭代拿到的仍是**旧的累加器**或**类型不一致的值**

<img src="/../img/assets_2025/image-20250307185524019.png" style="zoom:30%;" />





#### 案例1：按属性对Object进行分类

```javascript
var arr = [{ name: 'Q', age: 21 },{ name: 'I', age: 20 },{ name: 'Q', age: 20 }];
function groupBy(array, property) {
    return array.reduce((pre, cur) => {
        // 取 age 属性的值作为对象 pre 的 key 值
        var key = cur[property];
        // 在 pre 数组中以 key 值分类
        if (!pre[key]) {
            pre[key] = [];
        }
        pre[key].push(cur);
        // 『将 pre 传递』
        return pre;
    }, {});
}
var res = groupBy(arr, 'age');
```

#### 案例2：按序运行 `Promise`

注意：『<u>reduce 中 pre 的值需要在函数体中使用 `return pre` 返回</u>』

```javascript
let p1 = a => new Promise((resolve, reject) => {resolve(a * 1);});
let p2 = a => new Promise((resolve, reject) => {resolve(a * 2);});
let p4 = a => new Promise((resolve, reject) => {resolve(a * 4);});
let f3 = a => a * 3;
const promiseArr = [p1, p2, f3, p4];
processArray(promiseArr, 1).then((res) => {console.log(res)});   // 24

// 方法一：直接使用 then - pre.then(cur)
function processArray(arr, initialValue) {
    return arr.reduce((pre, cur) => pre.then(cur), Promise.resolve(initialValue));
}
// 方法二：使用 async/await（等价于方法一，但可读性/性能通常不如 pre.then(cur)）
function processArray(arr, initialValue) {
    return arr.reduce(async (prePromise, cur) => {
        const pre = await prePromise;
        return cur(pre);
    }, Promise.resolve(initialValue));
}

// 方法三：更推荐的写法（可读性最好）
async function processArray(arr, initialValue) {
    let result = initialValue;
    for (const item of arr) {
        result = await item(result);
    }
    return result;
}
```
- 注意：`async reducer` 的累加器从第二次迭代开始就是 Promise，所以要么像“方法一”那样用 `pre.then(cur)`，要么像“方法二”那样把参数命名为 `prePromise` 并在回调开头 `await prePromise`。

    ![image-20241023121506614](/../img/assets_2023/image-20241023121506614.png)

    ```js
    function processArray(arr, initialValue) {
        return arr.reduce(async (pre, cur) => {
            // 错误点：pre 从第二次迭代开始是 Promise，但这里没有 await pre
            return await cur(pre);
        }, initialValue);
    } // 可能得到 NaN / 类型错误（取决于 cur 期待的入参类型）
    ```

    <img src="/../img/assets_2023/image-20241023120944651.png" alt="image-20241023120944651" style="zoom:50%;" />

#### 案例3：使用reduce实现map

注意：『<u>reduce遍历数组时会忽略 empty，所以使用 `pre[index]`更准确</u>』

```js
// map - 一个由原数组每个元素执行回调函数的结果组成的新数组
var new_array = arr.map(function callback(currentValue[, index[, array]]) {
	...
}[, thisArg])
```

```javascript
[1, 2, , 3].mapUsingReduce(
  (currentValue, index, array) => currentValue + index + array.length
); // [5, 7, , 10]

Array.prototype.mapUsingReduce = function (callback, thisArg) {
    // this 就是调用该函数的数组：this instanceof Array =》 true
    return this.reduce(function (pre, cur, index, array) {
        // reduce 的第四个参数是调用数组本身
        pre[index] = callback.call(thisArg, cur, index, array);
        return pre;
    }, []);
};
```

🆚 『 <u>`reduce` / `filter` / `map` / `forEach` 都会跳过 `empty`（稀疏数组的洞）</u>』（但 `null`、`undefined`、`''` 这些“有值的元素”不会跳过）如下图

```js
Array.prototype.mapUsingReduce = function (callback, thisArg) {
    return this.reduce((pre, cur, index, array) => {
        // push 会把数组变成“致密数组”，丢失原数组的 empty 位置；用 pre[index] 才能保留洞位
        pre[index] = callback.call(thisArg, cur, index, array);
        return pre;
    }, [])
}// [5, 7, , 10]
```

![image-20241023165216325](/../img/assets_2023/image-20241023165216325.png)
