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

# React那些难懂的Hooks

## 背景

【开骂】垃圾 React 官网，hooks 描述只有晦涩难懂的几个词，用起来一踩一个坑，贼准

- 推荐学习链接

[React 开发者Dan的文章](https://overreacted.io/zh-hans/a-complete-guide-to-useeffect/)（推荐，React 开发者）、[React Hooks 最佳实践](https://juejin.cn/post/6844904165500518414)（云音乐的最佳实践）



## 重点摘要

推荐阅读：

- [如何在useEffect里做数据请求](https://www.robinwieruch.de/react-hooks-fetch-data/) // todo
- [如何处理错误和加载状态](https://www.robinwieruch.de/react-hooks-fetch-data/) // todo

关键点：

- **Effects 拿到的，总是定义它那次渲染中的 props 和 state**
- 如果想要读取最新值，最简单的方法是使用 refs，详细： [这篇文章的最后一部分](https://overreacted.io/how-are-function-components-different-from-classes/) // todo



## 详解

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
  });

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

当`setCount`的时候，React会**带着一个不同的`count`值再次调用组件**。然后，React会更新DOM以保持和渲染输出一致；

任意一次渲染中的`count`常量都不会随着时间改变。**渲染输出会变是因为我们的组件被一次次调用**。



### 2、每一次渲染都有它自己的事件处理函数

组件的函数每次渲染都会被调用，但是每一次调用中count值都是常量，并且它被赋予了当前渲染中的状态值；

每一次渲染都有一个**“新版本”的`handleAlertClick；`**

每一个版本的`handleAlertClick`“记住” 了**它自己的 `count`；**

事件处理函数**“属于”某一次特定的渲染**，当你点击的时候，它会使用那次渲染中counter的状态值。



### 3、每次渲染都有它自己的Effects

并不是count的值在“不变”的effect中发生了改变，而是**effect 函数本身在每一次渲染中都不相同**；

每一个effect版本“看到”的`count`值（props 和 state ）都来自于它属于的那次渲染；

React会记住你提供的effect函数，并且会在每次更改作用于DOM并让浏览器绘制屏幕后去调用它 -- **概念上，你可以想象effects是渲染结果的一部分**。



### 总结：

**在组件内什么时候去读取props或者state是无关紧要的**。因为它们不会改变。在单次渲染的范围内，props和state始终保持不变。

<img src="/../img/assets_2019/:Users:haoling:Library:Application Support:typora-user-images:image-20220822213608238.png" alt="image-20220822213608238" style="zoom:80%;" />



### 逆潮而动（从过去渲染中的函数读取未来的 props 和 state 而不是捕获的值）

使用 useRef

```react
function Example() {
  const [count, setCount] = useState(0);
  const latestCount = useRef(count); // ref

  useEffect(() => {
    latestCount.current = count;
    setTimeout(() => {
      console.log(`You clicked ${latestCount.current} times`); // 输出的是最新值
    }, 3000);
  });
  // ...
}
  
```



## 那Effect中的清理又是怎样的呢？

React只会**在浏览器绘制后运行effects**。这使得你的应用更流畅因为大多数effects并不会阻塞屏幕的更新；

Effect的清除同样被延迟了 --> **上一次的effect会在重新渲染后被清除**

案例：假设第一次渲染的时候`props`是`{id: 10}`，第二次渲染的时候是`{id: 20}`。

执行顺序：

1、**React 渲染`{id: 20}`的UI** 

--> 2、浏览器绘制，我们在屏幕上看到`{id: 20}`的UI 

-->  3、**React 清除`{id: 10}`的effect。**（第一次渲染中effect的清除函数只能看到`{id: 10}`这个props）

--> React 运行`{id: 20}`的effect



## 同步， 而非生命周期

React会根据我们当前的 props 和 state 同步到DOM。“mount”和“update”之于渲染并没有什么区别；

React 重要的是目的，而不是过程；如果我们的结果依赖于过程而不是目的，我们会在同步中犯错 --> **先渲染属性A，B再渲染C，和立即渲染C并没有什么区别**



### 如何好好使用依赖 - 依赖数组参数

**在不需要的时候避免调用effect** --> useEffect 提供了依赖数组参数(deps) --> 如果当前渲染中的这些依赖，和上次运行这个 effect 时一样，React 会自动跳过这次 effect

useEffect的设计意图就是要**强迫你关注数据流的改变**，然后决定我们的effects该如何和它同步，而不是忽视它直到我们的用户遇到了bug

两种诚实告知依赖的方法：

- 一般推荐告知 React 所有依赖项，避免遗漏导致变更后，后面的 effect 因为没有检测到变化而被跳过；

- 还有一种策略：**修改 effect 内部的代码，确保它包含的值只会在需要的时候发生变更** --> 不是告知错误的依赖，而是减少 effect 依赖



##### 一些移除依赖的小技巧：

###### 1、让Effects自给自足

案例[错误的依赖](https://codesandbox.io/s/q3181xz1pj?file=/src/index.js:303-330)： 将 count +1 改为 setState 的函数形式后，就不需要再依赖 count

```react
function Counter() {
  const [count, setCount] = useState(0);  
	useEffect(() => {
      const id = setInterval(() => {
        // setCount(count + 1);
        // --> 改为
        setCount(c => c + 1); // 目的是把 count 转换成 count + 1 再返回给 React；React 其实已经知道当前的 count 了
        // effect 并不需要知道当前的 count 值，React 知道就可以了
      }, 1000);
      return () => clearInterval(id);
    }, [
      // count  
    ]);
  return <h1>{count}</h1>;
}
```

###### 2、函数式更新 - 只在effects中传递最小的信息会很有帮助

案例[定时器](https://codesandbox.io/s/zxn70rnkx?file=/src/index.js)：当你想更新一个状态，并且这个状态更新依赖于另一个状态的值时，你可能需要用useReducer去替换它们

- reducer可以让你**把组件内发生了什么(actions)和状态如何响应并更新分开表述。**
- 之前渲染中的 reducer 能知道新的 props，因为 dispatch 的时候，React 记住的是 action，下一次渲染时再调用 reducer，新的props 就能被访问到，reducer 的调用不是在 effect 里面

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

假如不想在step改变后重启定时器：[使用 reducer](https://codesandbox.io/s/xzr480k0np)，effect不再关心怎么更新状态，它只负责告诉我们发生了什么

```react

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
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



###### 3、把函数移到Effects里 - 如果某些函数仅在effect中调用，你可以把它们的定义移到effect中

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

一个常见的误解是，“函数从来不会改变”，但是每一次渲染都在改变

几种方式移除依赖数组中的函数：

1、**如果一个函数没有使用组件内的任何值，你应该把它提到组件外面去定义，然后就可以自由地在effects中使用**

案例：移到组件外面后，不用再添加依赖，因为它们不在渲染范围内，因此不会被数据流影响

<img src="/../img/assets_2019/:Users:haoling:Library:Application Support:typora-user-images:image-20220823221249581.png" alt="image-20220823221249581" style="zoom:80%;" />

2、包装成 useCallback - 本质上是添加了一层依赖检查， **使函数本身只在需要的时候才改变，而不是去掉对函数的依赖**

- **useCallback 保证了如果一个函数的输入改变了，这个函数就改变了。如果没有，函数也不会改变。**
- **我想强调的是，到处使用`useCallback`是件挺笨拙的事。**当我们需要将函数传递下去并且函数会在子组件的effect中被调用的时候，`useCallback` 是很好的技巧且非常有用。

案例：如果 query 保持不变，getFetchUrl 也会保持不变，effect 就不会重新运行

<img src="/../img/assets_2019/:Users:haoling:Library:Application Support:typora-user-images:image-20220823221449906.png" alt="image-20220823221449906" style="zoom:90%;" />









