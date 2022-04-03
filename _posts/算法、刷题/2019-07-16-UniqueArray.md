---
layout:     post
title:      数组去重（算法整理）
subtitle:  
date:       2019-07-16
author:     
header-img: 
catalog: true
tags:
    - < 算法题 >
typora-root-url: ..
---

# 数组去重（算法整理）



### 方法一：哈希表思想

思路：新建一个结果数组、一个存储标志的对象，遍历数组，对象中`arr[i]`对应的值为`ture`的时候不重复添加

**for...of循环的i代表的是value（多用于数组），for...in循环的是key（多用于对象）**；

```javascript
function unique(arr) {
    var result = [];
    var hash = {};
    for (let elem of arr) { 
        if (!hash[elem]) {
            result.push(elem);
            hash[elem] = true; //作为一个是否在数组中存在的标志
        }
    }
    return result;
}
var arr = [1, 23, 34, 45, 1, 6, 4, 23];
unique(arr);//[1, 23, 34, 45, 6, 4]
```

### 方法二：遍历数组用indexOf() / includes()

思路1 - `indexOf`：新建一个数组 `result`，遍历原数组，当`arr[i]`不存在于新数组中，就添加 `push` 进新数组，得到的新数组就是不重复的。

```javascript
function unique(arr){
    let res = [];
    for(let item of arr) {
        if(res.indexOf(item) === -1) {
            res.push(item);
        }
    }
    return res
}
var arr = [1, 23, 34, 45, 1, 6, 4, 23];
unique(arr);//[1, 23, 34, 45, 6, 4]
```

思路2 - `includes`：新建一个数组`result`，如果新数组中该值`i`不存在，则将原数组内容添加`push(i)`。

**for...of循环的i代表的是value（多用于数组），for...in循环的是key（多用于对象）**；

```javascript
function unique(arr) {
    let result = [];
    for(let item of arr) {
        if(!res.includes(item)) {
            res.push(item);
        }
    }
    return result;
}
var arr = [1, 23, 34, 45, 1, 6, 4, 23];
unique(arr);//[1, 23, 34, 45, 6, 4]
```



### 方法三：数组下标判断法

思路1：利用 `indexOf` 返回的始终是第一个相同数值的下标，如果新增的内容在原来数组中的下标和现在的下标不同，那么就证明它是重复的

```javascript
function unique(arr){
    var result=[];
    for (var i = 0; i < arr.length; i++) {
        if(arr.indexOf(arr[i]) === i){ //下标不同，代表了重复
            result.push(arr[i]);
        }
    }
    return result;
}
var arr = [1, 23, 34, 45, 1, 6, 4, 23];
unique(arr);//[1, 23, 34, 45, 6, 4]
```

思路2（重复的直接删除，不保留）：利用indexOf和lastIndexOf，从前往后和从后往前找到的下标如果相同，就证明没有重复的值；

```javascript
function unique(arr){
    var result=[];
    for (var i = 0; i < arr.length; i++) {
        if(arr.indexOf(arr[i]) === arr.lastIndexOf(arr[i])){
            result.push(arr[i]);
        }
    }
    return result;
}
var arr = [1, 23, 34, 45, 1, 6, 4, 23];
unique(arr);//[34, 45, 6, 4]
```



### 方法四：排序，去除相邻的重复值

思路：给数组排序，排序后相同的值会相邻，遍历排序后的数组时，新数组只加入不与前一值重复的值。

```javascript
function unique(arr){
    arr.sort();
    var result=[arr[0]];
    for (var i = 1; i < arr.length; i++) {
        if(arr[i] !== result[result.length-1]){
            result.push(arr[i]);
        }
    }
    return result;
}
var arr = [1, 23, 34, 45, 1, 6, 4, 23];
unique(arr);//[1, 23, 34, 4, 45, 6]
```

新方法：使用reduce

```javascript
let arr = [1, 23, 34, 45, 1, 6, 4, 23];;
let result = arr.sort().reduce((init, current) => {
    if(init.length === 0 || init[init.length-1] !== current) {
        init.push(current);
    }
    // 这里的if也可以使用indexOf方法进行判断是否等于-1
    return init;
}, []);
console.log(result); // [1, 23, 34, 4, 45, 6]
```



### 方法五：双层循环遍历

思路1：有重复值时终止当前循环`i`，进入外层循环`i+1`；找到的是往后遍历不会有重复值的值；

```javascript
function unique(arr){
    var result=[];
    for (var i = 0; i < arr.length; i++) {
        for (var j = i+1; j < arr.length; j++) {
            if(arr[i]===arr[j]){
                ++i;
            }
        }
        result.push(arr[i]);
    }
    return result;
}
var arr = [1, 23, 34, 45, 1, 6, 4, 23];
unique(arr);//[34, 45, 1, 6, 4, 23]注意这里的顺序和上面的都不同
```

思路2：两层循环遍历，如果有重复的值，则用splice（）删除

```javascript
function unique(arr) {
    for (let i=0, len=arr.length; i<len; i++) {
        for (let j=i+1; j<len; j++) {
            if (arr[i] == arr[j]) {
                arr.splice(j, 1);
                // splice 会改变原数组长度，所以要将数组长度 len 和下标 j 减一
                len--;
                j--;
            }
        }
    }
    return arr
}
var arr = [1, 23, 34, 45, 1, 6, 4, 23];
unique(arr);//[1, 23, 34, 45, 6, 4]
```



### 方法六：ES6的Set结构（高性能）

思路：Set结构成员的值都是唯一的，Set函数可以接受一个数组作为参数，用来初始化。

```javascript
function unique(arr){
    var x = new Set(arr);
    return [...x]; // 这里直接返回x的话，结果是Set(6){1, 23, 34, 45, 6, 4}
}
// 或者简写成
function unique(arr){
    return [...new Set(arr)]
}
var arr = [1, 23, 34, 45, 1, 6, 4, 23];
unique(arr);//[1, 23, 34, 45, 6, 4]
```

```js
// 去除数组的重复成员
[...new Set(array)]
//上面的方法也可以用于，去除字符串里面的重复字符。
[...new Set('ababbc')].join('')
// "abc"
```

### 方法七：ES6的 `Array.filter()`

思路：利用`Array.filter()`方法的过滤方法，将数组进行过滤，筛选条件是数组下标与检索下标一致。

`concat() ` 方法用于连接两个或多个数组；该方法**不会改变现有的数组**，而仅仅会返回被连接数组的一个副本。

`filter()`方法通过检查指定数组中符合条件的所有元素；**不会改变原始数组**，创建一个新的数组。

```javascript
function unique(arr) {
    return arr.filter((item, index)=> {
        return arr.indexOf(item) === index
    })
}
var arr = [1, 23, 34, 45, 1, 6, 4, 23];
unique(arr);//[1, 23, 34, 45, 6, 4]
```



