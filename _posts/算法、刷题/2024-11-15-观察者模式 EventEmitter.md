---
layout:     post
title:      观察者模式 EventEmitter
subtitle:  
date:       2024-11-15
author:     
header-img: 
catalog: true
tags:
    - < 算法题 >
typora-root-url: ..
---

## 观察者模式 EventEmitter

 手动实现 `EventEmitter`

```js
// addListener(event, listener) 为指定事件添加一个监听器，默认添加到监听器数组的尾部。
// removeListener(event, listener) 移除指定事件的某个监听器，监听器必须是该事件已经注册过的监听器。它接受两个参数，第一个是事件名称，第二个是回调函数名称。
// once(event, listener) 为指定事件注册一个单次监听器，即 监听器最多只会触发一次，触发后立刻解除该监听器。
// emit(event, [arg1], [arg2], [...]) 按监听器的顺序执行执行每个监听器，如果事件有注册监听返回 true，否则返回 false。
//setMaxListeners(n) 默认情况下， EventEmitters 如果你添加的监听器超过 10 个就会输出警告信息。 
```

常见错误：

1、constructor：super()只在派生类中使用，这是定义的基类

<img src="/../img/assets_2023/image-20241204184401713.png" alt="image-20241204184401713" style="zoom:40%;" />

2、remove：如果 listener为null，删除整个 event 列表 + 注意： index>-1再删除

<img src="/../img/assets_2023/image-20241204185039520.png" alt="image-20241204185039520" style="zoom:35%;" />

<img src="/../img/assets_2023/image-20241214162410391.png" alt="image-20241214162410391" style="zoom:30%;" />

3、once：①简单的包裹函数实现执行+删除②由于 newLis 函数使用箭头函数定义，this = 外部this

<img src="/../img/assets_2023/image-20241204185322848.png" alt="image-20241204185322848" style="zoom:30%;" />

4、emit：①『<u>this.events[event] 每次都是在this重新查询，有once添加的函数时，会导致遍历 index 对不齐</u>』

<img src="/../img/assets_2023/image-20241204185654244.png" alt="image-20241204185654244" style="zoom:30%;" />

​	②记得对 emit 传入的参数进行处理 + 注意：即使 args 不是数组，可能是字符串等类型，还是要传递给函数的

<img src="/../img/assets_2023/image-20241210180150829.png" alt="image-20241210180150829" style="zoom:30%;" />



```js
// 写到第三遍，相对完善的写法
class EventEmitter {
    constructor() {
        this._max = 10;
        this.eventList = {};
    }
    addListener(event, listener) {
        if (!this.events[event]) {
            this.events[event] = [listener]
        } else {
            if (this.events[event].length > this.max) {
                throw Error('max number');
            }
            this.events[event].push(listener);
        }
    }
    removeListener(event, listener) {
        if (Array.isArray(this.events[event])) {
            // 删除某个类别的事件
            if(!listener) {
                delete this.events[event];
            } else {
                // 删除某个回调
                let index = this.events[event].indexOf(listener);
                index > -1 && this.events[event].splice(index, 1);
            }
        }
    }
    once(event, listener) {
        let onceFun = (...args) => {
            listener(...args);
            this.removeEventListener(event, onceFun);
        }
        this.addListener(event, onceFun);
    }
    emit(event) {
        if (!this.events[event]) {
            return false;
        } else {
            let static = [...this.events[event]]; // once导致 events动态变化造成异常
            let argsList = arguments.slice(1);
            static.forEach((it, index) => {
                let args = argsList[index];
                Array.isArray(args) ? it(...args) : it(args);
            })
        }
    }
    setMaxListeners(n) {
        this.max = n;
    }
}

// 测试
const emitter1 = new EventEmitter();
var onceListener = function (args) {
    console.log('我只能被执行一次', args, this);
}

var listener = function (args) {
    console.log('我是一个listener', args, this);
}

emitter1.once('click', onceListener);
emitter1.addListener('click', listener);

emitter1.emit('click', ['参数']);
emitter1.emit('click');

emitter1.removeListener('click', listener);
emitter1.emit('click');
```





