---
layout:     post
title:      Object和Map
subtitle:  
date:       2021-09-09
author:     
header-img: 
catalog: true
tags:
    - < JS原生基础相关 >
typora-root-url: ..
---

## Object和Map

参考博客：[](https://zhuanlan.zhihu.com/p/358378689)

【定义】

Map：保存键值对，能记住键的【原始插入顺序】，**【任何值（包括对象）】都可以作为键/值**；

Obecjt：键值对，**键为【字符串】**，值为任何值

|           | Object                                                       | Map                                                          |
| --------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 用法      | 1、**键的类型**：只能是字符串/Symbol；<br>2、顺序：不会完全保持插入时的顺序<br><img src="/../img/assets_2019/image-20210910181811510.png" alt="image-20210910181811510" style="zoom:30%;" /><br>3、计算长度绕：`Object.keys(testObj).length`<br>4、默认不可迭代，只支持 `for-in` 访问<br>（也可以使用对象方法访问键/值，例如：`Object.keys(o)、Object.values(o)、Object.entries(o)`）<br>`obj[Symbol.iterator] !== undefined; // false`<br/><img src="/../img/assets_2019/image-20210910200926462.png" alt="image-20210910200926462" style="zoom:30%;" /><br>5、可以覆盖原型上的键<br/>6、`JSON` 默认支持 `Object` | 1、**键的类型**：任何类型；<br/>2、保持其插入时的顺序；<br/>3、计算长度简单：`mapObj.size()`<br/>4、可迭代对象，支持 `for-of`/`forEach`<br>`map[Symbol.iterator] !== undefined; // true`<br/>5、不会覆盖原型上的键<br/>6、`JSON` 默认不支持 `Map` |
| 句法      | 1、创建<br>`const o = {}; // 对象字面量 `<br/>`const o = new Object(); // 调用构造函数`<br/>`const o = Object.create(null); // 调用静态方法 `<br/>2、新增/修改元素<br>`o.x=1 //属性名不能包含空格、标点符号，不能以数字开头 `<br>`o['y'] = 2;`<br>3、读取元素<br>`o.x ` 或者 `o['y']; `<br>或者使用 ES2020 新增的条件属性访问表达式来读取<br>` o?.x` 或者  `o?.['y']`<br>4、删除元素<br>`delete o.b;` | 1、创建：<br>`const map = new Map(); // 调用构造函数`<br>2、新增/修改元素<br>`map.set('x', 1); `<br/>3、读取元素<br/>`map.get('x');`<br/>4、删除元素<br/>` map.delete('b');` |
| 性能      |                                                              | Map性能比Object好（占用内存小，增删速度更快）                |
| 适合 场景 | 1、只是简单的数据结构时，选择 `Object`，因为它在数据少的时候占用内存更少，且新建时更为高效 <br/>2、需要用到 `JSON` 进行文件传输时，选择 `Object`，因为 `JSON` 不默认支持 `Map` <br/>3、需要对多个键值进行运算时，选择 `Object`，因为句法更为简洁 <br/>4、需要覆盖原型上的键时，选择 `Object` | 1、储存的键不是字符串/数字/或者 `Symbol` 时，选择 `Map`，因为 `Object` 并不支持 <br/>2、储存大量的数据时，选择 `Map`，因为它占用的内存更小 <br/>3、需要进行许多新增/删除元素的操作时，选择 `Map`，因为速度更快 <br/>4、需要保持插入时的顺序的话，选择 `Map`，因为 `Object` 会改变排序 <br/>5、需要迭代/遍历的话，选择 `Map`，因为它默认是可迭代对象，迭代更为便捷 |

【注释】：

##### 1、Object 中各属性排序规则（案例如上表）

- **非负整数** 会最先被列出，排序是从小到大的数字顺序
- 然后所有字符串，负整数，浮点数会被列出，顺序是根据插入的顺序
- 最后才会列出 `Symbol`，`Symbol` 也是根据插入的顺序进行排序的

##### 2、JSON 默认不支持 Map，但是可以转一层

若想要通过 `JSON` 传输 `Map` 则需要使用到 `.toJSON()` 方法，然后在 `JSON.parse()` 中传入复原函数来将其复原。详细可以看下 [JSON 的序列化和解析](https://link.zhihu.com/?target=https%3A//juejin.cn/post/6931927132427780103%23heading-5)

