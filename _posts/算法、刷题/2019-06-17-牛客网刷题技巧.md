---
layout:     post
title:      冒泡排序、插入排序、快排
subtitle:  
date:       2019-06-17
author:     
header-img: 
catalog: true
tags:
    - < 算法题 >
typora-root-url: ..
---



# 冒泡排序、插入排序、快排
数组:一组有序的数据,数据类型可以不一样。

数组的定义:2种方式

1. 通过**构造函数**创建数组/实例化Array对象

语法: `var arrname=new Array();` 创建一个**空数组**。

语法: `var arrname=new Array(num);` 创建一个数组**长度为num的数组**；其中每一项都是 `undefined`。

语法: `var arrname=new Array(n1,n2...);` 创建由**传入参数组成的一个数组**。

2. 通过**字面量**的方式创建数组/使用[]方式

语法：`var array=[];` **创建空数组**

###  冒泡排序（3种）

外面的循环控制的是比较的轮数；里面的循环控制的是每一轮比较的次数。

1.从前往后  注意：i取值的时候，不能取到`length-1`，否则j就会越界。

```javascript
var arr = [1, 9, 5, 2, 10, 3, 6, 4];// 待排序数组
for (var i = 0; i < arr.length - 1; i++) {
    for (var j = i + 1; j < arr.length; j++) {
        if (arr[i] > arr[j]) {
            var temp = arr[j];
            arr[j] = arr[i];
            arr[i] = temp;
        }
    }
}
```
2.从后往前  先通过循环确定最后一位的最大值，然后继续往前确定数值。

```javascript
var arr = [1, 9, 5, 2, 10, 3, 6, 4];// 待排序数组
for(var i = 0;i < arr.length - 1; i++){
    for(var j = 0;j < arr.length - 1 - i; j++){
        if (arr[j] > arr[j+1]) {
            var temp = arr[j];
            arr[j] = arr[j+1];
            arr[j+1] = temp;
        }
    }
}
```
3.优化冒泡：设置一个标志位，减少不必要的循环

当原数组是，`arr = [1,2,4,3];`在经过第一轮冒泡排序之后，数组就变成了`arr = [1,2,3,4];`

此时，数组已经排序完成了，但是按上面的代码来看，数组还会继续排序，所以我们加一个标志位，如果某次循环完后，没有任何两数进行交换，就将标志位 设置为 true，表示排序完成，这样我们就可以减少不必要的排序，提高性能。

```javascript
var arr = [3, 4, 1, 2];
function bubbleSort (arr) {
    var max = arr.length - 1;
    for (var j = 0; j < max; j++) {
        // 声明一个变量，作为标志位
        var done = true;
        for (var i = 0; i < max - j; i++) {
            if (arr[i] > arr[i + 1]) {
                var temp = arr[i];
                arr[i] = arr[i + 1];
                arr[i + 1] = temp;
                done = false;
            }
        }
        if (done) {
            break;
        }
    }
    return arr;
}
bubbleSort(arr);
```



### 插入排序  是冒泡排序的优化（2种）

**实现原理**：将一个数插入一个已经排好序的数据中。

1. 第一次循环时，**从第2个数开始处理**。我们将第1个数作为已经排好序的数据：当第2个数 > 第1个数时，将第2个数放在第1个数后面一个位置；否则，将第2个数放在第1个数前面。此时，前两个数形成了一个有序的数据。
2. 第二次循环时，我们处理第3个数。此时，**前两个数形成了一个有序的数据**：首先比较第3个数和第2个数，当第3个数 > 第2个数时，将第3个数放在第2个数后面一个位置**并结束此次循环**【优化点】；否则，再和第1个数比较。如果第3个数 > 第1个数，则将第3个数插入第1个数和第2个数中间；否则，第3个数 < 第1个数，则将第3个数放在第1个数前面。此时，前三个数形成了一个有序的数据。
3. 后续的数据同理处理，直至结束。

<img src="https://pic3.zhimg.com/v2-91b76e8e4dab9b0cad9a017d7dd431e2_b.webp" style="zoom:50%"  />

```javascript
// 实现一：两层循环，当前i与每个值都比较
var arr = [89, 56, 100, 21, 87, 45, 1, 88]; // 待排序数组
// 按照从小到大的顺序排列
// 外面这层循环是里层循环的次数
for(var i= 1; i < arr.length; i++){
  	// --j 表示从数组第i位逐个向前遍历比大小
    for(var j = i; j > 0; --j){
      	// 如果当前位小于前一位，则交换 --> 小的在前 --> 排序方式位：小->大
        if(arr[j-1] > arr[j]){
          	// 利用es6的变量交换：[a,b]=[b,a]
          	[arr[j-1], arr[j]]=[arr[j], arr[j-1]];
        }
    }
}
```

优化：**在 i 第一次比较不需要交换时，不需要再执行当前内层循环** -> 数组小于 i 的部分已经是有序数组了

```javascript
// 实现
var arr = [89, 56, 100, 21, 87, 45, 1, 88];
for(var i= 1;i<arr.length;i++){
  	// 优化：小于i的位置都已经排好序了，跳过当前 i 的循环
  	if(arr[i-1] < arr[i]) {
      	continue;
    }
    for(var j=i;j>0;--j){
        if(arr[j-1]>arr[j]){
          	[arr[j-1],arr[j]]=[arr[j],arr[j-1]];
        }
    }
}
```

### 快速排序 对冒泡排序的一种改进

又称：划分交换排序

**基本思想**：通过一趟排序将要排序的数据 **分割成独立的两部分**，其中一部分的所有数据比另一部分的所有数据要小，再按这种方法对这两部分数据分别进行快速排序，整个排序过程可以递归进行，使整个数据变成有序序列。

<img src="http://pic2.zhimg.com/v2-d4e5d0a778dba725091d8317e6bac939_b.webp"/>

```javascript
var quickSort = function (arr) {
  	// 退出递归的条件
    if (arr.length <= 1) { 
      	return arr; 
    }
    var midIndex = Math.floor(arr.length / 2);
    // 从原数组中拿出 mid 值
    var mid = arr.splice(midIndex, 1)[0];
    var left = [];
    var right = [];
  	// 以 mid 为基准，循环将原数组拆分成两个数组
    for (var i = 0; i < arr.length; i++) {
        if (arr[i] > mid) {
          	left.push(arr[i]);
        } else {
          	right.push(arr[i]);
        }
    }
  	// 递归 - 左半部分 + mid + 右半部分
   	return quickSort(left).concat([mid], quickSort(right));
};
```

