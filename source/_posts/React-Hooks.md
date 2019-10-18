---
title: React Hooks
date: 2019-10-18 15:02:15
tags:
---

## Hook 出现的原因

### 在组件之间复用状态很难
React并没有提供给组件附加复用逻辑的操作，比如状态管理中的把组件连接到store，解决这类问题方式一般是通过高阶组件或者render props；但是这样做会重新组织你的组件结构，代码不容易理解，可读性差，还会带来的问题是组件层级嵌套

在Hook中可以使用自定义Hook提取状态逻辑，在不修改组件状态的情况下复用状态逻辑

### 复杂的组件不容易理解
对于React中一个复杂的组件来说，每个生命周期中都有一些不相关的操作，componentDidMount 和 componentDidUpdate 有请求的逻辑，但是componentDidMount可能也有其它不相关的逻辑比如定时器或事件监听，而要在componentWillUnmount清除。相关联的代码被拆分开来，同一个生命周期中又组合一些不相关的代码
状态逻辑混杂，不容易拆分，所以很多人会将React结合状态管理库使用，但是这样会引入一些抽象的概念，需要在不同文件之间切换，使得复用变得更加困难

为了解决这个问题，Hook将组件中相关联的部分拆分成更小的函数，使用Effect Hook。

### class难以理解
class组件形式对于新手是不友好的，必须要掌握JavaScript的this的工作方式，开发者对函数组件和class组件的差异也存在分歧，甚至要区分函数组件和class组件的使用场景， 并且class 压缩不友好

Hook 可以在不适用class的情况下，使用更多React的特性。

## 基本使用
### State Hook 保存组件状态 useState
Demo: [地址](https://codesandbox.io/s/demo1-quvqp)
```jsx
import React, { useState } from "react";

function App() {
  const [count, setCount] = useState(0);
  return (
    <div className="App">
      <p>{count}</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
}
```
useState 需要的参数是一个初始化的值，返回值是一个数组，里面有一对值：第一个是state的值，第二个是更新state的函数

### Effect Hook 处理函数副作用 useEffect