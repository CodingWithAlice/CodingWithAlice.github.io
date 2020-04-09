---
layout:     post
title:     工作 vue template编译
subtitle:  
date:       2020-04-09
author:     
header-img: 
catalog: true
tags:
    - < 工作遇到的问题记录 >
typora-root-url: ..
---


# 工作 vue template编译

场景：在场景中遍历了一个数值，`v-for="item in (5 - data.num)"`，结果报错`RangeError: Invalid array length`

参考：

总结：发生原因是进行计算的时候`data.num`的值没有获得，解决方法是在外层加一个`v-if="data.num"`或者是将这个计算在异步拿到`data`值的时候进行计算。但是我想知道为什么在`v-for`中写简单的计算式会报错，直接写`data.num`是不会报错的，会等有值了再循环。

Vue的模板编译在$mount之后，进行编译最后生成render函数来生成虚拟DOM，虚拟DOM通过diff算法来更新DOM。

我看了很多博客，把自己对“编译”二字做的事情的理解输出一下。