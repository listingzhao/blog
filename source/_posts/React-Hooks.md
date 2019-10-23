---
title: React Hooks
date: 2019-10-18 15:02:15
tags:
---

## Hook 出现的原因

### 在组件之间复用状态很难

React 并没有提供给组件附加复用逻辑的操作，比如状态管理中的把组件连接到 store，解决这类问题方式一般是通过高阶组件或者 render props；但是这样做会重新组织你的组件结构，代码不容易理解，可读性差，还会带来的问题是组件层级嵌套

在 Hook 中可以使用自定义 Hook 提取状态逻辑，在不修改组件状态的情况下复用状态逻辑

### 复杂的组件不容易理解

对于 React 中一个复杂的组件来说，每个生命周期中都有一些不相关的操作，componentDidMount 和 componentDidUpdate 有请求的逻辑，但是 componentDidMount 可能也有其它不相关的逻辑比如定时器或事件监听，而要在 componentWillUnmount 清除。相关联的代码被拆分开来，同一个生命周期中又组合一些不相关的代码
状态逻辑混杂，不容易拆分，所以很多人会将 React 结合状态管理库使用，但是这样会引入一些抽象的概念，需要在不同文件之间切换，使得复用变得更加困难

为了解决这个问题，Hook 将组件中相关联的部分拆分成更小的函数，使用 Effect Hook。

### class 难以理解

class 组件形式对于新手是不友好的，必须要掌握 JavaScript 的 this 的工作方式，开发者对函数组件和 class 组件的差异也存在分歧，甚至要区分函数组件和 class 组件的使用场景， 并且 class 压缩不友好

Hook 可以在不适用 class 的情况下，使用更多 React 的特性。

## 基本使用

### State Hook 保存组件状态 useState

Demo: [地址](https://codesandbox.io/s/demo1-quvqp)

```jsx
import React, { useState } from 'react'

function App() {
    const [count, setCount] = useState(0)
    return (
        <div className="App">
            <p>{count}</p>
            <button onClick={() => setCount(count + 1)}>Click me</button>
        </div>
    )
}
```

useState 需要的参数是一个初始化的值，返回值是一个数组，里面有一对值：第一个是 state 的值，第二个是更新 state 的函数

### Effect Hook 处理函数副作用 useEffect

useEffect 可以看作是 React 生命周期 componentDidMount，componentDidUpdate 和 componentWillUnmount 三个函数的组合。

#### 是否需要清除

useEffect 可以返回一个函数，在这个函数中可以进行清除操作，比如取消订阅，清除计时器等；

-   默认情况下，effect 在每次渲染的时候都会执行，React 会在执行当前 effect 之前对上一个 effect 进行清除

Demo: [地址](https://codesandbox.io/s/demo2effecthook-6e2f6)

```jsx
import React, { useState } from 'react'

function App() {
    const [count, setCount] = useState(0)

    useEffect(() => {
        document.title = `You clicked ${count} times.`
    })

    useEffect(() => {
        console.log('useEffect')
        return () => {
            console.log('componentWillUnmount')
        }
    })
    return (
        <div className="App">
            <p>You clicked {count} times.</p>
            <button onClick={() => setCount(count + 1)}>Click me</button>
        </div>
    )
}
```

##### 注意点

-   使用多个 Effect 实现关注点分离，可以按照代码的用途分离，而不是像生命周期函数那样包含了各种代码

#### Effect 每次渲染都会执行的原因

主要的原因是这样的设计可以让我们在写发布订阅等类型的组件的时候少出 bug，在 class 组件中当 props 改变的时候，很多时候会忘记在 componentDidUpdate 中处理清除更新订阅的逻辑，这是导致的常见 bug，这样的设计 Effect 默认会处理更新逻辑，它会在调用一个新的 Effect 之前对上一个 Effect 进行清理。

#### 跳过 Effect 的默认行为进行性能优化

在某些情况下，每次都执行 Effect 可能会导致一些性能问题，解决的办法通常是添加判断逻辑来解决，useEffect 的第二个可选参数接收一个数组，React 会根据数组中的值对上一次的值进行比较，只有值更新了才会执行 effect，如果值相等，会跳过这个 effect。

```jsx
useEffect(() => {
    document.title = `You clicked ${count} times.`
}, [count])
```

##### 注意点

-   数组中包含了外部作用域会随时变化并且在 effect 中使用的变量
-   如果想执行只执行一次的 effect，可以传递一个 [] 空数组

### 自定义 Hook

在使用 Hook 复用状态逻辑之前，有两种方式是使用比较流行的，高阶组件和 render Props；使用 Hook 能在不新增组件的情况下实现状态复用；只需要把公共的逻辑提取到单独的函数中。

#### 注意点

-   自定义 Hook 是一个函数，函数名称必须以`use`开头，函数内部可以调用其它 Hook。
-   多个 Hook 之间可以通过参数传递信息，因为本身就是函数。

### useContext

#### context API

Demo: [地址](https://codesandbox.io/s/contextdemo-y8po5)

接收一个 context 对象并返回 context 的当前值。当前 context 的值由上层组件的`<MyContext.Provider>` 的 `value` props 决定。

```jsx
const value = useContext(MyContext)
```

useContext(MyContext) 相当于 class 组件中的 `static contextType = MyContext` 或者 `<MyContext.Consumer>` useContext 让我们能读取到 context 的值和订阅 context 的变化；需要我们在上层组件树中使用`<MyContext.Provider>`来为下层组件提供 context。

Demo: [地址](https://codesandbox.io/s/usecontextdemo-bgyzr)

### useCallback 记忆函数

useCallback 主要是用来缓存函数的，原因是因为在 class 组件中子组件如果有绑定的函数，可以把函数绑定在组件的 this 上，但是在 Hook 中不行；

```jsx
function App() {
    const handleClick = () => {
        console.log('click')
    }
    return <ChildenComponent onClick={handleClick}></ChildenComponent>
}
```

上面的函数组件渲染的时候，都会重新渲染子组件；有了 useCallback 的话就可以得到一个记忆后的函数；这样子只要子组件只要继承了 PureComponent 或者使用了 React.memo 就可以有效避免不必要的渲染。

```jsx
function App() {
    const memoizedHandleClick = useCallback(() => {
        console.log('click')
    }, [])
    return <ChildenComponent onClick={memoizedHandleClick}></ChildenComponent>
}
```

### useMemo 记忆值

useMemo 是用来性能优化的手段，避免一些在渲染阶段的开销大的计算；它和 useCallback 的区别是 useMemo 会在渲染阶段执行传入的函数，并把结果返回；而 useCallback 是把这个函数返回，达到缓存函数的目的。

```jsx
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b])
```

### useRef 引用对象

```jsx
const refContainer = useRef(initialValue)
```

useRef 返回的是一个可变的 ref 对象，它的.current 属性被初始化为传入的参数`initialValue`；ref 是一种咱们熟悉的访问 dom 的主要方式，如果把 useRef()的返回值以`<div ref={refContainer}>`形式传入组件，就可以通过 refContainer.current 值访问组件或真实的 dom 节点；

但是除了能访问 dom 节点外 useRef() 比 ref 更有用。它的.current 可以方便的保存任何可变值，类似于一个 class 的实例属性。

#### Hook Capture value 特性

Demo: [地址](https://codesandbox.io/s/capturevaluedemo-o408y)

```jsx
function App() {
    const [msg, setMsg] = useState('')
    const handleValueChange = e => {
        setMsg(e.target.value)
    }
    const handleSubmit = () => {
        setTimeout(() => {
            alert(msg)
        }, 2000)
    }
    return (
        <div className="App">
            <input value={msg} onChange={handleValueChange} />
            <button onClick={handleSubmit}>确定</button>
        </div>
    )
}
```

在上面的代码中，在输入框中输入值，然后点击确定之后再次修改输入框中的值，2 秒之后弹出的是修改之前的值；这就是 Capture Value 特性；而在 class 组件中 2 秒之后弹出的是修改之后的值，因为值是挂载在 class 实例上的。

使用`useRef`可以避免这样的问题

Demo: [地址](https://codesandbox.io/s/capturevaluedemo-o408y)

```jsx
function App() {
    const refMsg = useRef('')

    const handleValueChange = e => {
        refMsg.current = e.target.value
    }

    const handleSubmit = () => {
        setTimeout(() => {
            alert(refMsg.current)
        }, 2000)
    }
    return (
        <div className="App">
            <input onChange={handleValueChange} />
            <button onClick={handleSubmit}>确定</button>
        </div>
    )
}
```

### useImperativeHandle 透传 Ref

Demo: [地址](https://codesandbox.io/s/useimperativehandledemo-en3hw)

useImperativeHandle 可以让父组件访问到子组件的 ref 引用。

```jsx
import React, {
    useRef,
    useEffect,
    useImperativeHandle,
    forwardRef,
} from 'react'
function InputCom(props, ref) {
    const inputRef = useRef(null)

    useImperativeHandle(ref, () => ({
        focus: () => {
            inputRef.current.focus()
        },
    }))

    return (
        <div>
            <input ref={inputRef} />
        </div>
    )
}

InputCom = forwardRef(InputCom)

function App() {
    const inputRef = useRef(null)

    useEffect(() => {
        inputRef.current.focus()
    }, [])

    return (
        <div className="App">
            <InputCom ref={inputRef} />
        </div>
    )
}
```

### useLayoutEffect 同步执行渲染副作用

方法参数和 useEffect 一致，只是执行顺序不同，它会在所有 dom 变更之后同步调用 effect，可以使用它读取 dom 布局并更新触发重新渲染，在浏览器绘制之前，useLayoutEffect 内部的更新操作会被同步更新。

注意点
- 应该优先使用useEffect，只有出问题的时候再考虑使用useLayoutEffect，一避免阻塞浏览器渲染

Demo: [地址](https://codesandbox.io/s/uselayouteffectdemo-2yl4w)

```jsx
import React, { useEffect, useLayoutEffect } from "react";
function App() {
    useLayoutEffect(() => {
        console.log('useLayoutEffect')
        const box = document.querySelector('#box')
        console.log(box.getBoundingClientRect().width)
        box.innerHTML = 'bbb'
    }, [])
    useEffect(() => {
        console.log('useEffect')
    })
    return (
        <div className="App">
            <div id="box" className="boxStyle">
                aaa
            </div>
        </div>
    )
}
```
