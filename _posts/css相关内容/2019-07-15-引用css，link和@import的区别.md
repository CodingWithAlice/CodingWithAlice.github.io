---
layout:     post
title:     引用css，link和@import的区别
subtitle:  
date:       2019-07-15
author:     
header-img: 
catalog: true
tags:
    - < 面试题整理 >
typora-root-url: ..
---



# 引用css，link和@import的区别



```xml
<link rel="stylesheet" rev="stylesheet" href="CSS文件" type="text/css" media="all" />   
```

```xml
 @import url("CSS文件"); 
```

区别1：link是XHTML标签，除了加载CSS外，还可以定义[RSS](https://link.jianshu.com/?t=http://baike.baidu.com/link?url=kUew5P-VHsAgfZen83YoKmxqPaQLtGfzUchfFJ_nmTt414G3N8jdrN8S-Mz4jejHx6fQJ3UF02_JOiy2Y2WNX_)等其他事务；@import属于CSS范畴，只能加载CSS

区别2：link引用CSS时，在页面载入时同时加载；@import需要页面网页完全载入以后加载。

区别3：link是XHTML标签，无兼容问题；@import是在CSS2.1提出的，低版本的浏览器不支持。

区别4：ink支持使用Javascript控制DOM去改变样式；而@import不支持。

