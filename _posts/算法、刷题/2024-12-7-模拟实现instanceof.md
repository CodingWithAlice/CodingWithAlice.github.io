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
    - < LTN2 >
typora-root-url: ..
---

## 模拟实现instanceof

核心功能：检测对象原型链上是否有被检测函数的原型对象

常见错误：

- for…in 不合适->重点关注的是原型链上的原型对象匹配情况，而不是去比较属性名或者属性值

```js
function fakeInstanceof(target, constructor) {
    // let cur = target.__proto__;
    let proto = Object.getPrototypeOf(target);
    while (proto) { // 使用递归也可以
        if (proto === constructor.prototype) {
            return true
        }
        proto = Object.getPrototypeOf(proto);
    }
    return false
}
```

上述方案是使用 `while` 实现的，自己写的时候喜欢用递归

-  Object.getPrototypeOf() 静态方法返回指定对象的原型（即内部 [[Prototype]] 属性的值

```js
function fakeInstanceof(target, constructor) {
    let cur = target.__proto__;
    // 相等
    if (cur === constructor.prototype) {
        return true
    } else {
        // 不相等，但是可以继续找下一层
        if (cur.__proto__) {
            return fakeInstanceof(cur, constructor)
        }
    }
    // 都不相等，则为 false
    return false
}
```





