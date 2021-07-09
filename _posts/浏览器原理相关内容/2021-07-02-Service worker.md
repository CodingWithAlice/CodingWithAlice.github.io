---
layout:     post
title:     Service worker
subtitle:  
date:       2021-07-02
author:     
header-img: 
catalog: true
tags:
    - < 浏览器原理 >
typora-root-url: ..
---

# Service worker

参考文章：[浏览器架构](https://xie.infoq.cn/article/5d36d123bfd1c56688e125ad3)

定义/作用：一种可以 web 开发者控制缓存的技术

注意：Service worker 中的代码是运行在**渲染进程**中的

当访问一个 URL 开始时，**网络线程** 会根据域名检查是否有 Service worker 会处理当前地址的请求，如果有，则 **UI 线程** 会找到对应的 **渲染进程** 去执行 Service worker 的代码，而 Service worker 可以让开发者决定这个请求是从 **本地存储** 还是从 **网络** 中获取数据。

<img src="/../img/assets_2019/8c45c55d238b901239d0eb4bd40f2892.png" alt="img" style="zoom:43%;" />

