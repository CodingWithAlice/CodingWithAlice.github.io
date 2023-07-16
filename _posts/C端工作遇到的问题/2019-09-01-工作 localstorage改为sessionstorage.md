---
layout:     post
title:      localstorage改为sessionstorage
subtitle:  
date:       2019-09-01
author:     
header-img: 
catalog: true
tags:
    - < 工作遇到的问题记录 >
typora-root-url: ..
---

##  localstorage改为sessionstorage

​	`localStorage` 和 `sessionStorage` 一样都是用来 **存储客户端临时信息** 的对象。

​	他们均只能存储 <u>字符串类型</u> 的对象（虽然规范中可以存储其他原生类型的对象，但是目前为止没有浏览器对其进行实现）。

|                       | localStorage                       | sessionStorage                                    |
| --------------------- | ---------------------------------- | ------------------------------------------------- |
| 生命周期              | 永久<br/>除非用户手动在浏览器清除  | 当前窗口或标签页<br/>一旦关闭，存储的数据就被清空 |
| 不同浏览器            | 无法共享数据                       | 无法共享数据                                      |
| 相同浏览器 - 不同页面 | （页面属于相同域名和端口）可以共享 | 无法共享数据                                      |
| 相同浏览器 - 同源页面 | 可以共享                           | 可以共享                                          |

【补充：什么事同源】：

`http://www.test.com`

`https://www.test.com `（不同源，因为 **协议不同**）

`http://my.test.com`（不同源，因为 **主机名不同**）

`http://www.test.com:8080`（不同源，因为 **端口不同**）



`localStorage` 和 `sessionStorage` 使用时 **使用相同的API** ：

`localStorage` 和 `sessionStorage` 都继承于 `Storage`，提供了统一的 API 来访问和设置数据

|  API列表   | 入参                 |             功能             | 举例说明                                                     |
| :--------: | -------------------- | :--------------------------: | ------------------------------------------------------------ |
|   clear    | /                    | 清空存储中的所有本地存储数据 | `localStorage.clear();`                                      |
|  getItem   | 参数key              |    获取对应key的本地存储     | `localStorage.getItem('gjxm');`<br/>// 对象访问方式同样有效<br/>`localStorage.gjxm = '51129';`<br/>`localStorage.gjxm;// 51129 ` |
|    key     | 整数索引             |  返回对应本地存储中索引的键  | `localStorage.key(0);`                                       |
| removeItem | 参数key              |    删除对应本地存储的key     | `localStorage.removeItem('gjxm')；`                          |
|  setItem   | 两个参数，key和value | 如果不存在则添加，存在则更新 | `localStorage.setItem('gjxm', 'zzh');`                       |


**枚举** `localStorage` 的方法：

```javascript
for(var i=0; i < localStorage.length; i++){
    var name = localStorage.key(i);
    var value = localStorage.getItem(name);
}
```



 通过 `getItem` 或直接使用 `localStorage["key"]` 获取到的信息均为**实际存储的副本**。

```javascript
localStorage.key = {
    value1:"value1"
};
localStorage.key.value1='a';

//这里是无法对实际存储的值产生作用的，下面的写法也不可以：
localStorage.getItem("key").value1="a";
```

一篇写的比较详细的 [文章](http://www.111cn.net/wy/html5/85886.htm)

