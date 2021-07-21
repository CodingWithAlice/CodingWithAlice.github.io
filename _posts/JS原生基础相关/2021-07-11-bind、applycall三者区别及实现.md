---
layout:     post
title:      bind、apply/call三者区别及实现
subtitle:  
date:       2021-07-08
author:     
header-img: 
catalog: true
tags:
    - < JS原生基础相关 >
typora-root-url: ..
---

### bind、apply/call三者区别及实现

参考文章：[「干货」细说 call、apply 以及 bind 的区别和用法](https://segmentfault.com/a/1190000018017796)、MDN

#### call和apply

- 共同点： 改变函数执行时的上下文 - 将 **一个对象的方法（必须是函数）** 交给另一个对象来执行，并且是 <span style="color:red">**立即执行**</span> 的

- 区别：

    第一个参数 - `Function` 的调用者，将会指向这个对象。如果不传，则默认为全局对象 `window`

    第二个参数：
    	`call` - 接收任意个参数；

    ​	`apply` - 必须是数组或者类数组，它们会被转换成类数组，传入 `Function` 中

    ```js
    Function.call(obj,[param1[,param2[,…[,paramN]]]]);
    Function.apply(obj[,argArray]);
    ```

    举个例子：

    ```js
    function func (a,b,c) {}
    
    // ---- call ----
    func.call(obj, 1,2,3)
    // func 接收到的参数实际上是 1,2,3
    
    func.call(obj, [1,2,3])
    // func 接收到的参数实际上是 [1,2,3],undefined,undefined
    
    // ---- apply ----
    func.apply(obj, [1,2,3])
    // func 接收到的参数实际上是 1,2,3
    
    func.apply(obj, {
        0: 1,
        1: 2,
        2: 3,
        length: 3
    })
    // func 接收到的参数实际上是 1,2,3
    ```

##### call 和 apply 的使用场景

- 1/2、对象的继承

    ```js
    function superClass () {
        this.a = 1;
        this.print = function () {
            console.log(this.a);
        }
    }
    
    function subClass () {
        superClass.call(this); // 执行函数，this 继承了 superClass 的 print 方法和 a 变量
        this.print();
    }
    
    subClass();// 1
    ```

- 2/2、借用方法

    `domNodes` 可以应用 `Array` 下的所有方法
    
    ```js
    // slice() - 浅拷贝，返回一个新的数组对象
    let domNodes = Array.prototype.slice.call(document.getElementsByTagName("*"));
    ```
    获取数组中的最值
    ```js
    // 重要的不是 this 的绑定对象，而是 apply 将 array 的数组拆解了作为参数给 Math.max
    let max = Math.max.apply(null, array);
    let min = Math.min.apply(null, array);
    ```
    
    <img src="/../img/assets_2019/image-20210712095024074.png" alt="image-20210712095024074" style="zoom:40%;" />
    
    实现两个数组的合并
    
    ```js
    let arr1 = [1, 2, 3];
    let arr2 = [4, 5, 6];
    
    Array.prototype.push.apply(arr1, arr2);
    // 这里相当于把 arr2 作为 apply 的第二个参数，把 arr2 拆解了 -- arr1.push(4,5,6)
    // arr1 = [1, 2, 3, 4, 5, 6]
    ```

#### bind

```js
// 如果 bind 的第一个参数是 null 或者 undefined，this 就指向全局对象 window。
Function.bind(thisArg[, arg1[, arg2[, ...]]])
```

和 `apply/call` 比较：

- 相同点：改变函数体内的 `this` 指向
- 不同点：**bind 方法的返回值是函数，并且需要 <span style="color:red">稍后调用</span>，才会执行**

案例：

```js
function add (a, b) { return a + b;}
function sub (a, b) {return a - b;}

// add 的 this = sub 函数，add 并没有使用 this，所以还是 return a+b 的结果
add.bind(sub, 5, 3); // 这时，并不会返回 8
add.bind(sub, 5, 3)(); // 调用后，返回 8
```

```js
let button = document.getElementById('button');
let text = document.getElementById('text');
button.onclick = function() {
    alert(this.id); // 弹出'text'
}.bind(text); // 注意要用匿名函数 -> 箭头函数的不会创建自身的上下文， this 在定义时确定
```



#### 用 apply/call 实现bind

<img src="/../img/assets_2019/image-20210712163819701.png" alt="image-20210712163819701" style="zoom:40%;" />

```js
Function.prototype.newBind = function (context) {
    let self = this; // 调用 bind 的函数
    let args = [].slice.call(arguments, 1); // 调用 bind 时传入的参数
    return function() {
        // context -  newBind 的第一个参数，即 this 要指向的对象
        self.apply(context, [...arguments,...args]);
    }
}
```

