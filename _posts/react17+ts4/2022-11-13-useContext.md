---
layout:     post
title:     useContext
subtitle:  
date:       2022-11-13
author:     
header-img: 
catalog: true
tags:
    - < React17+TS4 >
typora-root-url: ..
---



# useContext

### Context - 无须明确地传遍每一个组件，就能将值深入传递进组件树

设计目的是为了共享那些对于一个组件树而言是“全局”的数据， 而不必显式地通过组建树逐层传递 props（在这之前，如果有全局状态管理需要使用 redux）

缺点：使组件的复用性变差

```react
// 下方案例直接从官方文档拷贝
const ThemeContext = React.createContext('light');
class App extends React.Component {
  render() {
    // 使用一个 Provider 来将当前的 theme 传递给以下的组件树。
    // 无论多深，任何组件都能读取这个值。
    // 在这个例子中，我们将 “dark” 作为当前的值传递下去。
    return (
      <ThemeContext.Provider value="dark">
        <Toolbar />
      </ThemeContext.Provider>
    );
  }
}

// 中间的组件再也不必指明往下传递 theme 了。
function Toolbar() {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

class ThemedButton extends React.Component {
  // 指定 contextType 读取当前的 theme context。
  // React 会往上找到最近的 theme Provider，然后使用它的值。
  // 在这个例子中，当前的 theme 值为 “dark”。
  static contextType = ThemeContext;
  render() {
    return <Button theme={this.context} />;
  }
}
```

### useContext 特性 （摘自 MDN） - 调用了 useContext 的组件，总会在 context 值变化时重新渲染

接收一个 context 对象（ `React.createContext`  的返回值），并返回该 context 的当前值；

当前的 context 值由 上层组件 中，距离当前组件最近的 `<MyContext.Provider>` 的 value prop 决定；

当 上层组件 最近的  `<MyContext.Provider>` 更新时，该 hook 会触发重渲染，并使用最新传递给 MyContext Provider 的 context value 值 --> 即使 祖先使用 memo 或者 shouldComponentUpdate，也会在组件本身使用 useContext 时重新渲染；

useContext(MyContext) 只是让你能够读取 context 的值 + 订阅 context 的变化，你仍需要在上层组件树中使用  `<MyContext.Provider>` 来为下层组件提供 context
