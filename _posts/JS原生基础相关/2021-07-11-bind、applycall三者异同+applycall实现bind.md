---
layout:     post
title:      bind、apply/call三者异同+apply/call实现bind
subtitle:  
date:       2021-07-08
author:     
header-img: 
catalog: true
tags:
    - < JS原生基础相关 >
    - < LTN1 >
typora-root-url: ..
---

### bind、apply/call三者异同+apply/call实现bind

##### 共同点

-  改变函数执行时的上下文 this

- 第一个参数 **如果不传，则默认为全局对象 `window`**


##### 区别

-  apply/call <span style="color:red">**立即执行**</span> ；bind 方法的返回值是函数，需要 <span style="color:red">**再次调用**</span>，才会执行

- 第二个参数：`call` - 接收任意个参数；`apply` - 必须是数组或者类数组

##### 使用场景：apply 和 Math.max() 

```js
// 重要的不是 this 的绑定对象，而是 apply 将 array 的数组拆解了作为参数给 Math.max
let max = Math.max.apply(null, array);
let min = Math.min.apply(null, array);
```

<img src="/../img/assets_2019/image-20210712095024074.png" alt="image-20210712095024074" style="zoom:40%;" />

##### 用 apply/call 实现bind

```js
var p = 1;
let a = function () {console.log(this.p + 1)}
a.fakeBind({ p: 100 })();
a.bind({ p: 8 })();

Function.prototype.fakeBind = function (context) {
    const func = this; // this 指向调用该原型函数的对象 - 也就是调用函数 a
    const args = [].slice.call(arguments, 1); // 『获取调用 fakeBind 时传入的参数』
    return () => func.apply(context, [...args, ...arguments]);
}
```

