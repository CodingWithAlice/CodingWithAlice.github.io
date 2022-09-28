---
layout:     post
title:      React那些难懂的Hooks
subtitle:  
date:       2022-08-16
author:     
header-img: 
catalog: true
tags:
    - < React >
typora-root-url: ..
---

# React那些难懂的Hooks - useEffect

## 背景

【开骂】垃圾 React 官网，hooks 描述只有晦涩难懂的几个词，用起来一踩一个坑，贼准

- 推荐学习链接

[React 开发者Dan的文章](https://overreacted.io/zh-hans/a-complete-guide-to-useeffect/)（推荐，React 开发者）、[React Hooks 最佳实践](https://juejin.cn/post/6844904165500518414)（云音乐的最佳实践）



## 重点摘要

推荐阅读：

- [如何在useEffect里做数据请求](https://www.robinwieruch.de/react-hooks-fetch-data/) 
- [如何处理错误和加载状态](https://www.robinwieruch.de/react-hooks-fetch-data/) 

关键点：

- **Effects 拿到的，总是定义它那次渲染中的 props 和 state**
- 如果想要读取最新值，最简单的方法是使用 refs，详细： [这篇文章的最后一部分](https://overreacted.io/how-are-function-components-different-from-classes/) 



## 详解案例

```react
function Counter() {
  const [count, setCount] = useState(0);
  
  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + count);
    }, 3000);
  }
  
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  },[count]);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
      <button onClick={handleAlertClick}>
        Show alert
      </button>
    </div>
  );
}
```

### 1、每一次渲染都有它自己的 Props and State

当`setCount`的时候，React会 **带着一个不同的`count`值再次调用组件**。然后，React 会更新 DOM 以保持和渲染输出一致；

任意一次渲染中的`count`常量都不会随着时间改变。**渲染输出会变是因为我们的组件被一次次调用**。



### 2、每一次渲染都有它自己的事件处理函数

组件的函数每次渲染都会被调用，但是每一次调用中 count 值都是常量，并且它被赋予了当前渲染中的状态值；

每一次渲染都有一个 **“新版本” 的 handleAlertClick 函数**

每一个版本的`handleAlertClick`“记住” 了**它自己的 `count`；**

**事件处理函数 “属于” 某一次特定的渲染**，当你点击的时候，它会使用那次渲染中 count 的状态值。



### 3、每次渲染都有它自己的 Effects

并不是 count 的值在“不变”的 effect 中发生了改变，而是 **effect 函数本身在每一次渲染中都不相同**；

每一个 effect 版本 “看到” 的`count`值（或者是 props 和 state ）都来自于它属于的那次渲染；

React 会记住你提供的 effect 函数，并且会在 <u>每次更改作用于 DOM 并让浏览器绘制屏幕后，去调用它</u> -- **概念上，你可以想象 effects 是渲染结果的一部分**。



### 总结：

**在组件内什么时候去读取 props 或者 state 是无关紧要的。因为它们不会改变。** 在单次渲染的范围内，props 和 state 始终保持不变。

<img src="/../img/assets_2019/:Users:haoling:Library:Application Support:typora-user-images:image-20220822213608238.png" alt="image-20220822213608238" style="zoom:80%;" />



### 逆潮而动（从过去渲染中的函数读取未来的 props 和 state 而不是捕获的值）

使用 useRef

```react
function Example() {
  const [count, setCount] = useState(0);
  const latestCount = useRef(count); // 使用 ref

  useEffect(() => {
    latestCount.current = count;
    setTimeout(() => {
      console.log(`You clicked ${latestCount.current} times`); // 输出的是最新值
    }, 3000);
  });
  // ...
}
```



## 那 Effect 中的清理又是怎样的呢？

React 只会 **在浏览器绘制后，运行 effects**。这使得你的应用更流畅，因为大多数 effects 并不会阻塞屏幕的更新；

Effect 的清除同样被延迟了 --> **上一次的 effect 会在重新渲染后，执行这次的 effects 前被清除**

案例：假设第一次渲染的时候`props`是`{id: 10}`，第二次渲染的时候是`{id: 20}`。

执行顺序：

1、**React 渲染`{id: 20}`的 UI** 

2、浏览器绘制，我们在屏幕上看到`{id: 20}`的 UI 

 3、**React 清除`{id: 10}`的 effect **

- 如何做到在新的渲染中清除上一次的数据：因为 <u>第一次渲染中 effect 的清除函数</u> 只能看到`{id: 10}`这个 props

4、React 运行`{id: 20}`的 effect



## React 核心是同步， 而非生命周期

React 会根据我们当前的 props 和 state 同步到 DOM。“mount” 和 “update” 这种生命周期之于渲染并没有什么区别；

**先渲染属性 A、B 再渲染 C，和立即渲染 C 并没有什么区别**

- React 重要的是目的，而不是过程；如果我们的结果依赖于过程而不是目的，我们会在同步中犯错  



### 如何好好使用依赖 - 依赖数组参数

**在不需要的时候避免调用effect** --> useEffect 提供了依赖数组参数(deps) --> 如果当前渲染中的这些依赖，和上次运行这个 effect 时一样，React 会 <u>自动跳过这次 effect</u>

useEffect 的设计意图就是要 **强迫你关注数据流的改变**，然后决定我们的 effects 该如何和它同步，而不是忽视它直到我们的用户遇到了 bug

两种诚实告知依赖的方法：

- 一般推荐告知 React 所有依赖项，避免遗漏导致变更后，后面的 effect 因为没有检测到变化而被跳过；

- 还有一种策略：**修改 effect 内部的代码，确保它包含的值只会在需要的时候发生变更** --> 不是告知错误的依赖，而是减少 effect 依赖



##### 一些移除依赖的小技巧：

###### 1、让Effects自给自足 -> 善用 setState(c => c+1) 这种函数形式

案例 [错误的依赖](https://codesandbox.io/s/q3181xz1pj?file=/src/index.js:303-330)： 将原本的 `count +1` 改为 `setState 的函数形式` 后，就不需要再依赖 count

```react
function Counter() {
  const [count, setCount] = useState(0);  
	useEffect(() => {
      const id = setInterval(() => {
        // setCount(count + 1); // 目的是把 count 转换成 count + 1 再返回给 React；
        // --> 改为
        setCount(c => c + 1); // 其实在这次渲染中 React 其实已经知道当前的 count 了
        // effect 并不需要知道当前的 count 值，React 知道就可以了
      }, 1000);
      return () => clearInterval(id);
    }, [
      // count  // 原本需要依赖 count，现在可以注释掉了
    ]);
  return <h1>{count}</h1>;
}
```

###### 2、函数式更新 - 只在effects中传递最小的信息会很有帮助（能理解，但是没实际用过）

当你想更新一个状态，并且这个状态更新依赖于另一个状态的值时，你可能需要用 useReducer 去替换它们

- reducer可以让你 **把 `组件内发生了什么` ( actions ) 和 `状态如何响应并更新` 分开表述。**
- 之前渲染中的 reducer 能知道新的 props，因为 dispatch 的时候，React 记住的是 action，下一次渲染时再调用 reducer，新的props 就能被访问到，reducer 的调用不是在 effect 里面

案例 [定时器](https://codesandbox.io/s/zxn70rnkx?file=/src/index.js)：假如不想在step改变后重启定时器：[使用 reducer](https://codesandbox.io/s/xzr480k0np)，effect不再关心怎么更新状态，它只负责告诉我们发生了什么

```react
function Counter() {
  const [count, setCount] = useState(0);
  const [step, setStep] = useState(1);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + step);
    }, 1000);
    return () => clearInterval(id);
  }, [step]); // 如果修改 step 会重启定时器

  return (
    <>
      <h1>{count}</h1>
      <input value={step} onChange={e => setStep(Number(e.target.value))} />
    </>
  );
}
```

改为：

```react

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState); // 在这里使用 reducer
  const { count, step } = state;

  useEffect(() => {
    const id = setInterval(() => {
      // dispatch了一个action来描述发生了什么
      dispatch({ type: 'tick' }); // 替代了 c => c + step， effect 和 step 状态解耦
    }, 1000);
    return () => clearInterval(id);
  }, [dispatch]);

  return (
    <>
      <h1>{count}</h1>
      <input value={step} onChange={e => {
        dispatch({
          type: 'step',
          step: Number(e.target.value)
        });
      }} />
    </>
  );
}

// 更新的逻辑全都交由reducer去统一处理
const initialState = {
  count: 0,
  step: 1,
};

function reducer(state, action) {
  const { count, step } = state;
  if (action.type === 'tick') {
    return { count: count + step, step };
  } else if (action.type === 'step') {
    return { count, step: action.step };
  } else {
    throw new Error();
  }
}
```



###### 3、把函数移到 effects 里 - 如果某些函数仅在 effect 中调用，你可以把它们的定义移到 effect 中

- 好处：不需要再考虑间接依赖

案例：这个情况其实很常见，尤其是初始化一个组件的时候

```react
useEffect(() => {
  fetchData();
}, []); 

// ---> 改为
useEffect(() => {
  function getFetchUrl() {
    return 'https://hn.algolia.com/api/v1/search?query=react';
  }
  async function fetchData() {
    const result = await axios(getFetchUrl());
    setData(result.data);
  }

  fetchData();
}, []); // ✅ Deps are OK
```



###### 3.1、有些函数不能移入 effect，我们仍旧不应该对依赖撒谎

**一个常见的误解是，“ 函数从来不会改变 ”**，但是每一次渲染都在改变，都是新的函数

几种方式移除依赖数组中的函数：

1、如果一个函数没有使用组件内的任何值，你应该 **把它提到组件外面去定义**，然后就可以自由地在effects中使用

案例：移到组件外面后，<u>不用再添加依赖</u>，因为它们不在渲染范围内，因此不会被数据流影响

<img src="/../img/assets_2019/:Users:haoling:Library:Application Support:typora-user-images:image-20220823221249581.png" alt="image-20220823221249581" style="zoom:80%;" />

2、包装成 useCallback - 本质上是添加了一层依赖检查， **使函数本身只在需要的时候才改变，而不是去掉对函数的依赖**

- **useCallback 保证了如果一个函数的输入改变了，这个函数就改变了。如果没有，函数也不会改变。**
- 当我们需要将函数传递下去，并且函数会在子组件的 effect 中被调用时，`useCallback` 是很好的技巧且非常有用；但是想强调的是，到处使用`useCallback`是件挺笨拙的事。

案例：如果 query 保持不变，getFetchUrl 也会保持不变，effect 就不会重新运行

<img src="/../img/assets_2019/:Users:haoling:Library:Application Support:typora-user-images:image-20220823221449906.png" alt="image-20220823221449906" style="zoom:90%;" />









