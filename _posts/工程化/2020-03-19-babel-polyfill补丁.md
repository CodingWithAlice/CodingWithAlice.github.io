---
layout:     post
title:     babel-polyfill 补丁
subtitle:  
date:       2020-03-19
author:     
header-img: 
catalog: true
tags:
    - < 工作遇到的问题记录 >
typora-root-url: ..
---


# babel-polyfill 补丁

场景：学习`babel`的时候，频繁看到 `polyfill`，俗称补丁

参考：各种网络博客

- 总结：

    `babel-polyfill` 是一种为低版本浏览器补充 ES6 方法/属性的模块

    作用： **为所有 API 增加兼容方法**

    需要配合 `preset-env` 使用

    缺点：需要在所有代码之前 `require`，且体积比较大



### 先区别一下前端常见的两个概念，`polyfill` 和 `shim`：

**`Polyfill`** --用于实现浏览器并不支持的原生 API 的代码（**没有提供新的 API，只是在 API 中实现缺少的功能**），用来为旧浏览器 **提供它没有原生支持的较新的功能**；使用时一般先 <u>检查当前浏览器是否支持某个 API</u>，如果不支持的话就加载对应的 `polyfill`，新旧浏览器就都可以使用这个 API 了

**`Shim`**—一般 **有自己的API**，而不是单纯实现原生不支持的 API。`shim` 是一个库，它将一个新的 API 引入到一个旧的环境中，而且仅靠旧环境中已有的手段实现，是一个优雅降级



### 再看babel的作用

已知 `babel` 用于将 es6 语法转换成浏览器普遍支持的 es5 语法，使开发能够使用便捷的 es6 进行开发。

但是babel只对es6语法进行转换，比如将箭头函数编译成函数形式，但是 **不会增加或修改原有的属性和方法**，比如 `Iterator`、`Generator`、`Set`、`Maps`、`Proxy`、`Reflect`、`Symbol`、`Promise` 等全局对象，以及一些定义在全局对象上的方法（比如 `Object.assign`）都不会转码。



### @babel/polyfill

如果要在低版本的浏览器中执行es6的方法，就需要 **对ES6函数进行补充**，安装并在页面引入 `@babel/polyfill`

**缺点**

- `polyfill` 是通过改变 **全局变量** 来兼容 ES6 的所有方法，给很多类的原型链上做了修改，会 **污染全局** 环境
- 这个插件是一个整体，把所有方法都加在原型链上，**体积很大**

解决方法：

- 1、配置 `@babel/preset-env` 只打包被使用函数的 `polyfill`

![image-20200319142328933](/../img/assets_2019/image-20200319142328933.png)

- 2、使用 `babel-plugin-transform-runtime`

    

### @babel/preset-env — 按需编译和打补丁

有点复杂，贴个[官网的配置](https://babeljs.io/docs/en/babel-preset-env) 

碰到过的配置：

`targets` 属性项目描述支持的运行环境 `{chrome:67}` 支持的浏览器最低版本

<img src="/../img/assets_2019/image-20200319150342134.png" alt="image-20200319150342134" style="zoom:67%;" />

<img src="/../img/assets_2019/ preset-env-targets.png" alt="image-20200319150234975" style="zoom:20%;" />

`useBuiltIns` 属性描述如何处理 `polyfill/usage` 只打包项目中使用到的函数的polyfill

<img src="/../img/assets_2019/image-20200319150801286.png" alt="image-20200319150801286" style="zoom:67%;" />



### @babel/transform-runtime - 解决代码重复出现

- 问题

    在默认情况下，`Babel` 会在每个输出文件中内嵌这些依赖的辅助函数的代码，如果多个源代码文件都依赖这些辅助函数，那么这些辅助函数的代码将会 **重复出现很多次**，造成代码冗余

- 该插件优点

    为了不让这些辅助函数的 **代码重复出现**，使用 `transform-runtime` 这个插件，就可以通过 `require` 导入依赖为代码中使用到的 ES6 API注入 `polyfill`

- 在使用 `babel-plugin-transform-runtime` 的时候必须把 `babel-runtime` 当做依赖



### babel-runtime

内部集成了：

- `core-js`： 转换一些内置类 (`Promise`, `Symbols`等等) 和静态方法 (`Array.from` 等)。绝大部分转换是这里做的。自动引入。
- `regenerator`： 作为 `core-js` 的拾遗补漏，主要是 `generator/yield` 和 `async/await` 两组的支持。当代码中有使用 `generators/async` 时自动引入
- `helpers`：在代码中有内置的 `helpers` 使用时，移除定义，并插入引用，如 `asyncToGenerator` ,  `jsx`, `classCallCheck` 等等，可以查看 [babel-helpers](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fbabel%2Fbabel%2Fblob%2F6.x%2Fpackages%2Fbabel-helpers%2Fsrc%2Fhelpers.js)

