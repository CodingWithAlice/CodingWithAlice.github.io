---
layout:     post
title:     onLoad & onReady
subtitle:  
date:       2021-08-09
author:     
header-img: 
catalog: true
tags:
    - < 浏览器原理 >
typora-root-url: ..
---

# onload & onready

##### 浏览器加载的步骤：

1. 解析html结构；
2. 加载外部 js 脚本和样式表文件；（预扫描）
3. 解析并执行 js 脚本；
4. dom树构建完成 - html 解析完毕（完成后触发 `onready` -> 即 `DOMContentLoaded`）；
5. 加载图片等外部文件（完成后触发 图片 `onload`）；
6. 页面加载完毕（完成后触发 页面 `onload`）




- DOMContentLoaded 和 onload 的区别：

    |          | DOMContentLoaded                              | onload                                              |
    | -------- | --------------------------------------------- | --------------------------------------------------- |
    | 执行时机 | 在 `html` **解析完毕** 后执行                 | 在页面 **所有元素（包括图片+页面）加载完成** 后执行 |
    | 执行次数 | onready可以执行多个，并且都可以按顺序得到执行 | 只执行最后一个                                      |

    

- DOMContentLoaded / onready

    定义：当纯HTML被完全加载以及解析时，会被触发，而不必等待样式表，图片或者子框架完成加载

    ```js
    document.addEventListener('DOMContentLoaded', (event) => {
        console.log('DOM fully loaded and parsed'); // 译者注："DOM完全加载以及解析"
    });
    ```
    
- onload

    定义：在页面或图像加载完成后立即发生

    ```html
    <!-- html中 -->
    <body onload="SomeJSCode"></body>
    ```

    ```js
    // js 中
    window.onload = function() {console.log('SomeJSCode')}
    ```

    

拼多多笔试题：

<img src="/../img/assets_2019/image-20210810104925684.png" alt="image-20210810104925684" style="zoom:30%;" />

考察点：

​	defer - 在 html 解析完成后执行，在触发 `DOMContentLoaded` 事件前执行 、 DOMContentLoaded 、 onload

浏览器输出：

<img src="/../img/assets_2019/image-20210810105049885.png" alt="image-20210810105049885" style="zoom:90%;" />
