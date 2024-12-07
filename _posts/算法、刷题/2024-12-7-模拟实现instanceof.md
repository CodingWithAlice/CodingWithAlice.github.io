---
layout:     post
title:      模拟实现instanceof
subtitle:  
date:       2024-12-7
author:     
header-img: 
catalog: true
tags:
    - < 算法题 >
    - < LTN1 >
typora-root-url: ..
---

## 模拟实现instanceof

核心功能：检测对象原型链上是否有被检测函数的原型对象

常见错误：

- for…in 不合适->重点关注的是原型链上的原型对象匹配情况，而不是去比较属性名或者属性值

```js
function fakeInstanceof(target, constructor) {
    let proto = Object.getPrototypeOf(target);
    // let proto = target.__proto__;
    while (proto) { // 使用递归也可以
        if (proto === constructor.prototype) {
            return true
        }
        proto = Object.getPrototypeOf(proto);
    }
    return false
}
```







