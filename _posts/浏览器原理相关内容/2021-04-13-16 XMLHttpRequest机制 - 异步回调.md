---
layout:     post
title:     16 XMLHttpRequest机制 - 异步回调
subtitle:  
date:       2021-04-13
author:     
header-img: 
catalog: true
tags:
    - < 浏览器原理 >
typora-root-url: ..
---


# 16 XMLHttpRequest机制 - 异步回调

## XMLHttpRequest解决的问题
在 XMLHttpRequest 出现之前，如果服务器数据有更新，依然需要**重新刷新整个页面**；

XMLHttpRequest 提供了**从 Web 服务器获取数据**的能力，如果你想要更新某条数据，只需要通过 XMLHttpRequest 请求服务器提供的接口，就可以获取到服务器的数据，然后再操作 DOM 来更新页面内容，整个过程只需要 **更新网页的一部分** 就可以了，而不用像之前那样还得刷新整个页面，这样既有效率又不会打扰到用户。


## 补充：同步回调VS异步回调
#### 定义
同步回调：回调函数 callback 是 **在主函数返回之前执行** 的，举例如下；
```js
let callback = function(){
    console.log('i am do homework')
}
function doWork(cb) {
    console.log('start do work')
    cb()
    console.log('end do work')
}
doWork(callback)
```
异步回调：回调函数 **在主函数外部执行**，举例如下；
```js
let callback = function(){
    console.log('i am do homework')
}
function doWork(cb) {
    console.log('start do work')
    // 在 doWork 函数执行结束后，又延时了 1 秒再执行
    setTimeout(cb,1000)   
    console.log('end do work')
}
doWork(callback)
```

#### 回调函数的调用栈
同步回调：在当前主函数的上下文中执行回调函数
异步回调：一般有两种方式

-   第一种是把异步函数做成一个任务，添加到 **信息队列尾部**；
-   第二种是把异步函数添加到 **微任务队列** 中，这样就可以在当前任务的末尾处执行微任务了

## XMLHttpRequest 运作机制
#### 流程图
<img src="/../img/assets_2019/image-20210413110259377.png" alt="image-20210413110259377" style="zoom:50%;" />

#### 发起XHR的简易请求代码
```js
function GetWebData(URL){
    // 1: 新建 XMLHttpRequest 请求对象
    let xhr = new XMLHttpRequest()
	// 2: 注册相关事件回调处理函数 
    xhr.onreadystatechange = function () {
        switch(xhr.readyState){
          case 0: // 请求未初始化
            break;
          case 1://OPENED
            break;
          case 2://HEADERS_RECEIVED
            break;
          case 3://LOADING  
            break;
          case 4://DONE
            if(this.status == 200||this.status == 304){
                console.log(this.responseText);
            }
            break;
        }
    }
    xhr.ontimeout = function(e) { console.log('ontimeout') }
    xhr.onerror = function(e) { console.log('onerror') }
 
	//3: 打开请求
    xhr.open('Get', URL, true);// 创建一个 Get 请求, true - 采用异步
 
     // 4: 配置参数
    xhr.timeout = 3000 // 设置 xhr 请求的超时时间
    xhr.responseType = "text" // 设置响应返回的数据格式
    xhr.setRequestHeader("X_TEST","time.geekbang")
 
	// 5: 发送请求
    xhr.send();
}
```
### 代码实现步骤
#### 第一步：创建 XMLHttpRequest 对象
`let xhr = new XMLHttpRequest()`：JS创建一个 `XMLHttpRequest` 对象xhr，用来执行实际的网络请求操作

#### 第二步：为 xhr 对象注册回调函数
网络请求比较耗时，注册回调函数后，后台执行完成后调用回调函数来同步执行结果。

XMLHttpRequest回调函数类型：

-   `ontimeout`，用来监控超时请求，如果后台请求超时了，该函数会被调用； 
-   `onerror`，用来监控出错信息，如果后台请求出错了，该函数会被调用；
-   `onreadystatechange`，用来监控后台请求过程中的状态，比如可以监控到 HTTP 头加载完成的消息、HTTP 响应体消息以及数据加载完成的消息等

#### 第三步：配置基础的请求信息+参数
`xhr.open('Get', URL, true); // 异步` 通过 open 接口配置一些基础的请求信息，包括请求的地址、请求方法（是 `get` 还是 `post`）和请求方式（同步还是异步请求）

也可以通过 xhr 内部属性类配置一些其他可选的请求信息，具体可以看上述代码

#### 第四步：发起请求
`xhr.send();`：

如顶部流程图所示，**渲染进程会将请求发送给网络进程**，然后网络进程负责资源的下载，等网络进程接收到数据之后，就会 **利用 IPC 来通知渲染进程**；

**渲染进程接收到消息之后，会将 xhr 的回调函数封装成任务并添加到消息队列** 中，等主线程循环系统执行到该任务的时候，就会根据相关的状态来调用对应的回调函数

## XMLHttpRequest 使用中的“坑”

1、跨域问题

2、HTTPS 混合内容的问题：

​	HTTPS 混合内容是 HTTPS 页面中包含了不符合 HTTPS 安全要求的内容，比如包含了 HTTP 资源，通过 HTTP 加载的图像、视频、样式表、脚本等，都属于混合内容