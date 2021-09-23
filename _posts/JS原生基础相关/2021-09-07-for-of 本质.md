---
layout:     post
title:      for-of本质
subtitle:  
date:       2021-09-07
author:     
header-img: 
catalog: true
tags:
    - < JS原生基础相关 >
typora-root-url: ..
---

## for-of 本质

参考博客：[讲可迭代对象的文章](https://juejin.cn/post/6844903573973630989)

1、`for..of` 循环首先会调用对象上的方法属性 `[Symbol.iterator]`——即调用 `range[Symbol.iterator]()`，得到一个包含 `next` 方法的对象

- 这个包含 `next` 方法的对象称为 **迭代对象（iterator object）**
- 属性 `range[Symbol.iterator]` 被称为**迭代对象生成器**或**迭代对象生成函数**

2、接下来， `for..of` 就是完全在跟这个迭代对象打交道了

3、每次 `for..of` 循环一次，就要调用一次 `next` 方法

4、从 `next` 方法返回的对象中（`{ done: ..., value: ... }`），我们能获得当前遍历的值（`value`）以及遍历是否结束的标记（`done`）

```js
// 1. for..of 循环首先会调用对象上的 [Symbol.iterator] 属性——range[Symbol.iterator]()，
range[Symbol.iterator] = function() {
    // 2. 接下来, for..of 就是完全在跟这个迭代对象打交道了
    return {
        current: this.from,
        last: this.to,
        // 3. 每次 for..of 循环一次，就要调用一次 next 方法
        next() {
            // 4. 从 next 方法返回的对象中，能获得当前遍历的值（value）以及遍历是否结束的标记（done）
            if (this.current <= this.last) {
                return { done: false, value: this.current++ }
            } else {
                return { done: true }
            }
        }
    }
}
```



------

for...in循环有几个缺点：

- 数组的键名是数字，但是for...in循环是 **以字符串作为键名** “0”、“1”、“2”等等
- for...in循环不仅遍历数字键名，还会遍历手动添加的其他键，甚至包括原型链上的键
- 某些情况下，for...in循环会以任意顺序遍历键名

总之，**`for...in`循环主要是为遍历对象而设计的，不适用于遍历数组**。



`for...of`循环相比上面几种做法，有一些显著的优点：

- 有着同for...in一样的简洁语法，但是没有 `for...in` 那些缺点。
- 不同用于forEach方法，它可以与 `break`、`continue` 和 `return` 配合使用。
- 提供了遍历所有数据结构的统一操作接口。

```js
for (var n of fibonacci) {
    if (n > 1000)
        break;
    console.log(n);
}
```





无法中途跳出`forEach`循环，break命令或return命令都不能奏效。
