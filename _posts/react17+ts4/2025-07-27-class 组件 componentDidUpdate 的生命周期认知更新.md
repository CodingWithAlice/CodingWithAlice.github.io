---
layout:     post
title:    class 组件 componentDidUpdate 的生命周期认知更新
subtitle:  
date:       2025-07-27
author:     
header-img: 
catalog: true
tags:
    - < React17+TS4 >
typora-root-url: ..
---



# class 组件 componentDidUpdate 的生命周期认知更新

总结：

1、class 组件中，componentDidUpdate 生命周期 **在 第一次 render 后才会执行**

2、class 组件中，render 可以直接通过 props/state 变化触发

3、class 组件中，想要阻塞 render ，只有 shouldComponentUpdate 生命周期



##### 问题

- umi max 框架中，使用 dva 管理数据，一个组件，如果 dva 拿到的数据变换了， componentDidUpdate 中只要不满足逻辑，就不会执行 setState ，就应该不会触发 render 吧 -> 答案：错误

##### 实际流程

- Dva 状态变化 → 组件 props 变化 → **优先触发一次 render** → 再执行 componentDidUpdate
    - componentDidUpdate 中是否调用 setState，只影响是否会二次触发 render，不影响**第一次因 props 变化导致的 render**

```js
// Dva 的状态通过 connect 映射到组件的 props
@connect(({ user }) => ({
    userName: user.name // dva 的 user 变化会导致 props.useName 变化
}))
```

##### 知识点

class 组件的 render 是否执行，不取决于 componentDidUpdate 中是否调用 setState

- 取决于 **props 是否变化**（dva 或者父组件）
- 取决于自身 **state 是否变化**（setState触发）
- **阻止 render** 只能通过定义 shouldComponentUpdate 返回 false

```js
shouldComponentUpdate(nextProps){
    return nextProps.userName !== this.props.userName // 手动控制
}
```

