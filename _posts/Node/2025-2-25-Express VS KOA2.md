---
layout:     post
title:    Express VS KOA2
subtitle:  
date:       2025-2-25
author:     
header-img: 
catalog: true
tags:
    - < node >
typora-root-url: ..
---



# Express VS KOA2

总结：

- Express 适合 **快速开发**（开箱即用） 和对 **功能完整性** 有较高要求的项目，内置了常用工具和功能
- Koa2 更适合 **对性能和代码灵活性** （轻量级）有较高要求的项目，尤其是处理 **复杂异步逻辑** 的场景



相同点：①Node ②路由功能 ③中间件机制

- 1、**都基于 NodeJS平台之上**：都利用 Node.js 的事件驱动、非阻塞 I/O 模型，能够高效处理大量并发请求，适用于构建高性能的 Web 应用和 API 服务
- 2、**都具备路由功能**：根据不同的 HTTP 请求方法 + URL 路径，将请求分发到相应的处理函数
- 3、**都使用中间件机制处理请求和响应**



不同点：①异步处理方式 ②中间件执行流程 ③错误处理 ④上下文

- **1、不同的异步处理方式：回调 VS async/await**

    Express：早期使用 **回调函数** 处理异步操作，容易出现回调地狱；现已支持 Promise **async/await**

    KOA2：原生支持 **async/await** ，异步操作更加简洁直观，避免了回调地狱

- **2、中间件执行流程：线性流程 VS 洋葱模型**

    Express：中间件执行遵循 **线性流程**，请求进入后依次经过每个中间件，执行完一个中间件后再进入下一个，若要将控制权传递给下一个中间件，需要调用 `next()` 函数

    KOA2：采用 **洋葱模型**，请求进入后依次经过每个中间件的 “入栈” 处理，到达最内层中间件后，再依次经过 “出栈” 处理。中间件可以在 `next()` 前后执行不同的代码逻辑

- 3、错误处理：

    Express：错误处理中间件需要显式地定义四个参数 `(err, req, res, next)`，当中间件中抛出错误时，会跳过正常的中间件，直接进入错误处理中间件

    ```js
    app.use((err, req, res, next) => {
        res.status(500).send('Error');
    });
    // 传递错误方式 - 将错误传递给错误处理中间件
    next(err);
    ```

    KOA2：错误处理可以通过 `try...catch` 结合中间件实现，在 `async` 中间件中捕获错误并进行处理，代码更加简洁

    ```js
    app.use(async (ctx, next) => {
        try {
            await next();
        } catch (err) {
            ctx.status = err.statusCode || 500;
            ctx.body = { error: err.message };
        }
    });
    // 传递错误方式 - 将错误传递给错误处理中间件
    ctx.throw(400, '参数错误');

- **4、上下文的使用：req+res VS ctx**

    Express：使用 req 和 res 处理请求和响应，

    KOA2：使用 ctx 上下文对象来封装请求和响应





