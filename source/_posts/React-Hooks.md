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

useEffect可以看作是React生命周期componentDidMount，componentDidUpdate 和 componentWillUnmount 三个函数的组合。

#### 是否需要清除
useEffect可以返回一个函数，在这个函数中可以进行清除操作，比如取消订阅，清除计时器等；
- 默认情况下，effect在每次渲染的时候都会执行，React会在执行当前effect之前对上一个effect进行清除

Demo: [地址](https://codesandbox.io/s/demo2effecthook-6e2f6)
```jsx
import React, { useState } from "react";

function App() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times.`;
  });

  useEffect(() => {
    console.log("useEffect");
    return () => {
      console.log("componentWillUnmount");
    };
  });
  return (
    <div className="App">
      <p>You clicked {count} times.</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
}
```
##### 注意点
- 使用多个Effect实现关注点分离，可以按照代码的用途分离，而不是像生命周期函数那样包含了各种代码

#### Effect每次渲染都会执行的原因
主要的原因是这样的设计可以让我们在写发布订阅等类型的组件的时候少出bug，在class组件中当props改变的时候，很多时候会忘记在componentDidUpdate中处理清除更新订阅的逻辑，这是导致的常见bug，这样的设计Effect默认会处理更新逻辑，它会在调用一个新的Effect之前对上一个Effect进行清理。

#### 跳过Effect的默认行为进行性能优化
在某些情况下，每次都执行Effect可能会导致一些性能问题，解决的办法通常是添加判断逻辑来解决，useEffect的第二个可选参数接收一个数组，React会根据数组中的值对上一次的值进行比较，只有值更新了才会执行effect，如果值相等，会跳过这个effect。

```jsx
useEffect(() => {
    document.title = `You clicked ${count} times.`;
}, [count]);
```
##### 注意点
- 数组中包含了外部作用域会随时变化并且在effect中使用的变量
- 如果想执行只执行一次的effect，可以传递一个 [] 空数组

