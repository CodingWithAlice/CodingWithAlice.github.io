---
layout:     post
title:     工作 vue template编译
subtitle:  
date:       2020-04-09
author:     
header-img: 
catalog: true
tags:
    - < vue2相关知识点 >
typora-root-url: ..
---


# 工作 vue template编译

场景：在场景中遍历了一个数值，`v-for="item in (5 - data.num)"`，结果报错`RangeError: Invalid array length`

参考：

总结：发生原因是进行计算的时候`data.num`的值没有获得，解决方法是在外层加一个`v-if="data.num"`或者是将这个计算在异步拿到`data`值的时候进行计算。但是我想知道为什么在`v-for`中写简单的计算式会报错，直接写`data.num`是不会报错的，会等有值了再循环。

​	Vue的模板编译在生命周期`$mount`之后，进行编译`compiler`(`parse`、`optimize`、`generate`)最后生成`render`函数来生成虚拟DOM，虚拟DOM通过diff算法来更新DOM。

​	我看了很多博客，把自己对“编译”二字做的事情的理解输出一下。

编译过程主要分成三步：

+ parse函数：（template解析）解析字符串型的template，通过对不同的html内容递归进行正则匹配（vue标签、属性、文本、指令等），截取满足条件的字符串，**转化成AST抽象语法树**

  - vue中截取规则：通过判断模板中`html.indexof(’<’)`的值来确定要截取的是标签还是是文本

    等于 0：			注释、条件注释、doctype、开始标签、结束标签中的某一种

    大于等于 0：	文本、表达式

    小于 0：			表示 html 标签解析完了，可能会剩下一些文本、表达式
    
[规则详细参考这篇博客]: https://blog.csdn.net/wang729506596/article/details/90947583

[和这篇博客]: https://juejin.im/post/5d6a347ee51d453b7779d581


+ optimize函数：对AST进行静态内容的优化，**标记静态节点**（那些和数据没关系，不需要每次更新）


    + 原因：为了之后dom diff时，diff算法会直接跳过静态节点，减少比、过程，优化patch的性能
    
    + 标准：不断递归，判断一个父级元素是静态节点，需要判断所有子节点都是静态节点；

  简单来说，没有使用vue独有的语法的节点就可以称为静态节点。

+ generate函数：**递归转化AST，创建 render 函数字符串**

  - render函数就是返回`_c(‘tagName’,data,children)`的方法（参数1:标签名，参数2:他的一些数据，属性/指令/表达式/方法等，属性3:当前标签的子标签，每个子标签的格式也是`_c(‘tagName’,data,children)`）

​        这个编译过程会在**第一次mount**的时候被调用生成**Vnode（描述节点的对象**，描述如何创建真实的DOM节点，有注释节点、文本节点、元素节点、组件节点、函数式组件、克隆节点）--Vnode作用是新旧vnode进行对比，只更新发生变化的节点，更新到真的DOM上。

Vnode更新节点的方法--patch（在原有的基础上修改DOM节点，渲染视图，可以增删改）