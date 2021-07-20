---
layout:     post
title:     Tree-shaking原理
subtitle:  
date:       2021-07-20
author:     
header-img: 
catalog: true
tags:
    - < webpack >
typora-root-url: ..

---

# Tree-shaking原理

参考文章：[Tree-Shaking性能优化实践 - 原理篇](https://juejin.cn/post/6844903544756109319)

核心要点记录：

- 什么是 `Tree-shaking`

    虽然依赖了某个模块，但其实只使用其中的某些功能。通过 tree-shaking，将没有使用的模块摇掉，这样来达到 **<span style="color:red">删除无用（没有用到的）js代码</span>** 的目的

- `Tree-shaking` 的前提条件

    <img src="/../img/assets_2019/image-20210720083823462.png" alt="image-20210720083823462" style="zoom:60%;" />

    ES6模块依赖关系是确定的，**和运行时的状态无关**，可以进行可靠的 **<span style="color:red">静态分析</span>**，这就是tree-shaking的基础。

    - 所谓静态分析就是不执行代码，**从字面量上对代码进行分析**，ES6之前的模块化，比如我们可以动态 `require` 一个模块，只有执行后才知道引用的什么模块，这个就不能通过静态分析去做优化。

