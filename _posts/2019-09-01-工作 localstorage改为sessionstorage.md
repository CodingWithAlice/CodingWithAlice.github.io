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

## 工作 localstorage改为sessionstorage

​	`localStorage`和`sessionStorage`一样都是用来**存储客户端临时信息**的对象。

​	他们均只能存储字符串类型的对象（虽然规范中可以存储其他原生类型的对象，但是目前为止没有浏览器对其进行实现）。

​	`localStorage`生命周期是永久，这意味着除非用户显示在浏览器提供的UI上清除`localStorage`信息，否则这些信息将永远存在。`sessionStorage`生命周期为当前窗口或标签页，一旦窗口或标签页被永久关闭了，那么所有通过`sessionStorage`存储的数据也就被清空了。

​	**不同浏览器无法共享**`localStorage`或`sessionStorage`中的信息。相同浏览器的不同页面间可以共享相同的` localStorage`（页面属于相同域名和端口），但是不同页面或标签页间无法共享`sessionStorage`的信息。这里需要注意的是，页面及标 签页仅指顶级窗口，如果一个标签页包含多个`iframe`标签且他们属于同源页面，那么他们之间是可以共享`sessionStorage`的。

`http://www.test.com
https://www.test.com `（不同源，因为**协议不同**）
`http://my.test.com`（不同源，因为**主机名不同**）
`http://www.test.com:8080`（不同源，因为**端口不同**）
`localStorage`和`sessionStorage`使用时**使用相同的`API`**：

`localStorage`和`sessionStorage`都继承于`Storage`，提供了统一的`api`来访问和设置数据。

​	

|   API列表    |                                                          | 举例说明                                                     |
| :----------: | :------------------------------------------------------: | ------------------------------------------------------------ |
|    clear     |               清空存储中的所有本地存储数据               | `localStorage.clear();`                                      |
|  `getItem`   |          接受一个参数key，获取对应key的本地存储          | `localStorage.getItem('order');`<br/>// 对象访问方式同样有效<br/>`localStorage.order = 'b110';`<br/>`localStorage.order; `// b110 |
|     key      |       接受一个整数索引，返回对应本地存储中索引的键       | `localStorage.key(0);`                                       |
| `removeItem` |          接受一个参数key，删除对应本地存储的key          | `localStorage.removeItem('order')；`                         |
|  `setItem`   | 接受两个参数，key和value，如果不存在则添加，存在则更新。 | `localStorage.setItem('order', 'a109');`                     |

`localStorage.setItem("key","value");`//以“key”为名称**存储一个值**“value”
`localStorage.getItem("key");`//**获取**名称为“key”的值
**枚举**`localStorage`的方法：

```javascript
for(var i=0; i < localStorage.length;i++){
     var name = localStorage.key(i);
     var value = localStorage.getItem(name);
}
```

**删除**`localStorage`中存储信息的方法：
`localStorage.removeItem("key");`//删除名称为“key”的信息。
`localStorage.clear();`//清空`localStorage`中所有信息
 通过`getItem`或直接使用`localStorage["key"]`获取到的信息均为**实际存储的副本**。

```javascript
localStorage.key = {value1:"value1"};
localStorage.key.value1='a';
//这里是无法对实际存储的值产生作用的，下面的写法也不可以：
localStorage.getItem("key").value1="a";
```

一篇写的比较详细的文章：

`http://www.111cn.net/wy/html5/85886.htm`