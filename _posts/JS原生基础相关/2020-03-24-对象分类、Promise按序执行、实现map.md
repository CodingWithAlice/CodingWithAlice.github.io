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
    - < LTN1 >
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



##### 案例1：按属性对Object进行分类

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

##### 案例2：按序运行 `Promise`

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
// 方法二：使用 async/await - cur(await pre)
function processArray(arr, initialValue) {
    return arr.reduce(async (pre, cur, index) => {
        return cur(await pre);
    }, initialValue);
}
```
- 注意：使用 `async/await` 的时候需要注意，`reduce` 在每次执行函数后，期望返回一个值（ `async` 会导致 `pre` 返回为一个 `pending` 的 `Promise`），以便进行下一步迭代：

    ![image-20241023121506614](/../img/assets_2023/image-20241023121506614.png)

    ```js
    function processArray(arr, initialValue) {
        return arr.reduce(async (pre, cur, index) => {
            return await cur(pre); // 错误点 - pre从第二个函数开始变成异步函数返回的非数字类型
        }, initialValue);
    } // 得到 NaN
    ```

    <img src="/../img/assets_2023/image-20241023120944651.png" alt="image-20241023120944651" style="zoom:50%;" />

##### 案例3：使用reduce实现map

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

🆚 『 <u>`reduce` （`filter`、`map`、`forEach`）会跳过 `empty` 的数组项</u>』（`null`、`undefined`、`’‘` 都不会跳过）如下图

```js
Array.prototype.mapUsingReduce = function (callback, thisArg) {
    return this.reduce((pre, cur, index, array) => {
        let callbackValue = cur ? callback.call(thisArg, cur, index, array) : null;
        pre.push(callbackValue);
        return pre;
    }, [])
}// [5, 7, 10] - 注意第三项没有被处理
```

![image-20241023165216325](/../img/assets_2023/image-20241023165216325.png)
