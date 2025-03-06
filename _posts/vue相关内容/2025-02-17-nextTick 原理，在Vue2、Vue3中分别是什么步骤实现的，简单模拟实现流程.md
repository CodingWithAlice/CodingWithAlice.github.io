---
layout:     post
title:     nextTick 原理，在Vue2、Vue3中分别是什么步骤实现的，简单模拟实现流程
subtitle:  
date:       2025-02-17
author:     
header-img: 
catalog: true
tags:
    - < vue相关知识点 >
typora-root-url: ..
---

# nextTick 原理，在Vue2、Vue3中分别是什么步骤实现的，简单模拟实现流程

nextTick

- 作用：在下次 DOM 更新循环结束之后，执行延迟回调

##### Vue2

利用 JavaScript 的异步执行机制（如微任务、宏任务），将传入 nextTick 的回调函数延迟到 DOM 更新后执行

   - 原因：因为 Vue 的数据更新是异步执行的，当你修改数据时，Vue 不会立即更新 DOM，而是将这些更新操作放入一个队列中，等到「下一个事件循环」时再「批量更新」 DOM，所以需要 nextTick 来确保在 DOM 更新后再执行某些操作
        - 实现步骤：
        - 1、回调函数收集：nextTick 会将传入的回调函数收集到一个队列中
        - 2、异步执行方式选择：Vue 2 会根据不同的浏览器环境选择合适的异步执行方式，优先使用微任务（如 Promise、MutationObserver），如果不支持微任务则使用宏任务（如 setTimeout）
        - 3、触发异步执行：当第一次调用 nextTick 时，会触发异步执行，将队列中的回调函数在下次 DOM 更新循环结束后依次执行

##### Vue3

原理与 Vue2类似，但是使用 queueMicrotask 来统一处理微任务
- 实现步骤：
       - 1、回调函数收集：nextTick 会将传入的回调函数收集到一个队列中
       - 2、使用 queueMicrotask 将执行的回调函数放入微任务，确保在 DOM 更新后使用

```js
//  Vue2 中的 nextTick 模拟实现
const callbacks = []; 
let pending = false;  // 标记是否正在执行回调函数
function flushCallbacks(){ // 异步执行回调函数
    pending = true; // 正在执行回调函数
    const copies = callbacks.slice(0);
    callbacks.length = 0;
    copies.forEach(it => it());
    pending = false;
}

// step2：根据当前浏览器环境，选择异步执行方式
let timeFunc;
if(typeof Promise !== 'undefined'){
    const p = Promise.resolve(); // 使用 Promise 作为微任务
    timeFunc = () => {
        p.then(flushCallbacks);
    }
} else if(typeof MutationObserver !== 'undefined') {
    const obs = new MutationObserver(flushCallbacks); // 使用 MutationObserver 作为微任务
    let count = 1;
    const textNode = document.createTextNode(String(count));
    obs.observe(textNode, { characterData: true }); // 监听目标节点的文本内容变化
    timeFunc = () => {
        count = count + 1;
        textNode.data = String(count);
    }
} else if(typeof setImmediate !== 'undefined'){
    timeFunc = () => {
        setImmediate(flushCallbacks); // 使用 setImmediate 作为宏任务
        // 在 Node.js 的事件循环中(在一些浏览器中也有支持)，setImmediate 的回调函数会在 I/O 回调之后、setTimeout 和 setInterval 的回调之前执行。它的执行时机是在当前轮次的事件循环结束后，下一轮事件循环开始时
    }
} else {
    timeFunc = () => {
        setTimeout(flushCallbacks, 0); // 使用 setTimeout 作为宏任务
    }
}

// step3: nextTick 函数 - Vue2 有第二个参数 content
function nextTick(cb, ctx) {
    let _resolve;
    // step1：存储回调函数的队列
    callbacks.push(() => {
        if(cb){ 
            try { cb.call(ctx) } catch(e){ console.error(e) }
        } else if(_resolve) {
            _resolve(ctx);
        }
    })
    // step4：执行异步任务
    if(!pending){
        timeFunc();
    }
    // 如果没有传入回调函数，返回一个 Promise
    if (!cb && typeof Promise !== 'undefined') {
        return new Promise(resolve => {
            _resolve = resolve;
        })
    }
}

// 使用示例
new Vue({
    data() {
        return { msg: 'hello' }
    },
    mounted() {
        this.msg = 'world';
        // 在 DOM 更新后执行回调
        this.$nextTick(() => {
            console.log('DOM 已经更新');
        });
    }
})
```

```js
// Vue3 中的 nextTick 模拟实现
const queue = []; // 存储回调函数的队列
let isFlushing = false; // 标记是否正在执行回调函数

// 执行队列中的回调函数
function flushJobs() {
    isFlushing = true;
    let job;
    while ((job = queue.shift())) {
        job();
    }
    isFlushing = false;
}

// nextTick 函数
function nextTick(cb) {
    return new Promise((resolve) => {
        const runner = () => {
            if (cb) {
                try { cb() } catch (e) { console.error(e) }
            }
            resolve();
        };
        // step1：存储回调函数的队列
        queue.push(runner);
        // step2：执行异步任务
        if (!isFlushing) {
            queueMicrotask(flushJobs);
        }
    });
}

// 使用示例
import { createApp, nextTick } from 'vue';
const app = createApp({
    data() {
        return { message: 'Hello'};
    },
    mounted() {
        this.message = 'World';
        nextTick(() => { console.log('DOM 已经更新') })
    }
});

app.mount('#app');
```





