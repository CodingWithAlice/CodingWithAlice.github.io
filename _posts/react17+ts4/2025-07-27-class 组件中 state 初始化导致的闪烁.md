---
layout:     post
title:    class 组件中 state 初始化导致的闪烁
subtitle:  
date:       2025-07-27
author:     
header-img: 
catalog: true
tags:
    - < React17+TS4 >
typora-root-url: ..
---



# class 组件中 state 初始化导致的闪烁

总结：

1、class 组件声明的 state 如果来源于 props 可以直接赋值，避免闪烁

2、`value ?? ‘’` 和 `value || ‘’` 的区别是 **`??` 只关心 `null` 和 `undefined`(2个)**，**|| 会过滤所有的 `falsy` 值(6个)**

- 即 `value` 是 `null` 或 `undefined` 时，`??` 和 `||` 都会判断为 `true` 返回为 `‘’`
- `value` 是 `false、0、“”、NaN` 时，`??` 会判断为`false`，依旧返回 `value` 的值；`||` 判断为 `true`，返回为 `‘’`



##### 问题

class 组件 state 初始化、更新后 值会闪烁

##### 原因

```js
class Input {
    constructor(props){
        super(props)
        this.state = {
            vslue: '' // 应该写成 props.value ?? ''
        }
    }
}
```
