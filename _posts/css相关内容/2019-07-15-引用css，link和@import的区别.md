---
layout:     post
title:     引用css，link和@import的区别
subtitle:  
date:       2019-07-15
author:     
header-img: 
catalog: true
tags:
    - < CSS相关内容 >
    - < LTN1 >
typora-root-url: ..
---



# 引用css，link和@import的区别

总结：link 在 『**性能**（预加载）』和 『**兼容性**（不止可以加载CSS、XHTML标签无兼容问题、js可修改样式）』上面优于 @import

- link 适用于自己写的/需要动态修改的样式；@import 适用于第三方样式/ 公共基础库

| 区别     | link                                                | @import【不建议】                  |
| -------- | --------------------------------------------------- | ---------------------------------- |
| 加载内容 | 除加载CSS外，还可以定义RSS、rel等其他事务           | 只能加载CSS（导入样式表）          |
| 加载顺序 | 在页面载入时，html+link 同时/并行加载（**预加载**） | 页面完全解析以后加载               |
| 兼容问题 | 是 XHTML 标签（无兼容问题）                         | CSS2.1提出（低版本的浏览器不支持） |
| DOM      | 支持JS控制DOM修改样式                               | DOM无法修改                        |

CSS 的四种引入方式：1、行内样式；2、内嵌样式；3、link【推荐】；4、@import

```html
<head>
    <!-- 3、link -->
    <link rel="stylesheet" rev="stylesheet" href="CSS文件" type="text/css" media="all" />
    <!-- 2、内嵌样式 -->
    <style type="text/css">
        .text {
            color: pink;
        }
    </style>
    <!-- 4、@import -->
    <style type="text/css">
         @import url("CSS文件"); /* 或者直接在 css 文件头导入 */
    </style>
</head>
<!-- 1、行内样式 -->
<div style="color: pink"></div> 
```

