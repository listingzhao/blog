---
title: React 生命周期
date: 2019-10-15 13:59:21
tags:
---

### 16 之前旧版生命周期

#### 加载阶段

-   constructor() 加载时调用一次，可以在这定义 this.state
-   getDefaultProps() 设置默认的 props ，也可以用 dufaultProps 设置
-   getInitialState() 初始化 state
-   componentWillMount() 组件加载时调用一次，整个生命周期只调用一次，可以修改 state
-   render() 创建虚拟 dom，diff 差异，更新 dom
-   componentDidMount() 组件渲染之后调用一次

#### 更新阶段

-   componentWillreceiveProps(nextProps) 组件接收到新的 props 调用
-   shoundComponentUpate(nextProps, nextState) 组件接收到新的 props 或 新的 state 时调用，返回值为 bool，false 能阻止组件更新
-   componentWillUpdate(nextProps, nextState) 只有组件将要更新时调用
-   render()
-   componentDidUpdate() 组件更新完成之后调用

#### 卸载阶段

-   componentWillUnmount() 当组件从 DOM 中移除时会调用

### 16 之后新版生命周期

组件的生命周期分为两个阶段，3 个运行时期
阶段 1： Render 阶段
纯净不包含副作用的，可能被 react 终止，暂停，或重新启动
阶段 2：Commit 阶段
可以使用 DOM, 运行副作用，安排更新。

#### 挂载时

-   constructor()
-   static getDerivedStateFromProps() 在调用 render 方法之前调用，并且在初始挂载及后续更新时都会被调用。它应返回一个对象来更新 state，如果返回 null 则不更新任何内容
-   render()
-   componentDidMount()

#### 更新时

触发更新 render 的方式有 new props ， setState()， forceUpdate()

-   static getDerivedStateFromProps()
-   shouldComponentUpdate() 根据 bool 返回值判断组件是否受到 props 和 state 更新的影响，首次渲染和 forceUpdate 时不会调用改方法，应该尽量使用 PureComponent, PureComponent 会对 props 和 state 进行浅层比较, 如果数据结构比较复杂是引用类型的话，PureComponent 会出现不被更新的问题，解决方案是使用不可变数据
-   render()
-   getSnapshotBeforeUpdate() 最近一次渲染输出之前调用，使得能在组件更新之前从 dom 中捕获一些信息，例如滚动位置；返回值将作为 componentDidUpdate()的参数
-   componentDidUpdate(prevProps, prevState, snapshot) 组件更新完成调用，首次渲染不会调用，可以在此处对 dom 进行操作，如果对更新前后的 props 进行了比较，也可以进行网络请求。

#### 卸载时

-   componentWillUnMount() 在组件卸载及销毁之前调用

#### 错误处理

-   componentDidCatch()
-   getDerivedStateFromError()

### 新旧版本对比

根据 react 官方文档的描述，componentWillMount，componentWillreceiveProps，componentWillUpdate 三个生命周期将在未来 17 版本中移除，所以应该尽量避免在新版本中使用，取而代之的是两个新的生命周期：getDerivedStateFromProps， getSnapshotBeforeUpdate；

componentWillMount 这个生命周期之前一般能做的事情是在这里做一些状态的赋值操作，或者是异步请求，但是状态的赋值可以放在 constructor 里，在这里异步请求的话也不能第一次 render 的时候渲染出数据

componentWillreceiveProps 的功能可以使用 getDerivedStateFromProps 配合 componentDidUpdate 来完成；使用的场景 1：组件有 newprops 的时候更新 state，getDerivedStateFromProps 是一个 static 方法，所以不能内部使用 this; 场景 2：当 props 改变的时候执行一些回调

关于 getDerivedStateFromProps
只有当 state 的值在任何时候都取决于 props 的时候才建议使用该生命周期; 否则导致多余的派生状态，这样会导致代码冗余，不好维护。

-   如果要根据 props 的变更执行一些操作，应该使用 componentDidUpdate
-   如果想在 props 变更的时候重新计算一些数据
-   如果想在 props 变更的时候重置一些状态，应该使用完全受控组件或有 key 的非可控组件

需要注意：

-   不要直接复制 props 的值到 state 中，应该实现一个受控组件，然后在父组件中合并两个值
-   对于不受控组件，如果想在 props 变化是重置 state；1. 重置所有 state，使用 key 属性 2. 仅更改某些字段的话，观察特殊属性的变化 3. 使用 ref 调用实例方法

### 为什么组件的生命周期会变动

React16 版本引入了异步渲染架构 Fiber，由于它主要功能是在组件渲染完成之前，可以中断渲染任务，中断之后组件剩下的生命周期不再执行，而是从头开始执行生命周期

### 为什么需要异步渲染

React16 之前对于虚拟 dom 的渲染是同步的，将每次更改的操作收集起来，和真实 dom 对比出变化之后，统一一次更新 dom 树的结构，原来的 stack reconciler 是一个递归执行的函数，父组件调用子组件 reconciler 的过程就是一个递归的过程，如果组件的的层级比较深，dom 树比较庞大，这个递归遍历的过程时间会比较长，这个时间段内，浏览器主线程被占用，其它的交互和渲染都会被阻塞，就会出现卡住的感觉

### 怎么实现的异步渲染

React16 版本引入了全新的异步渲染架构 Fiber，主要对 reconciler 阶段进行了拆分，以后进行详细的探究

### Fiber 带来的问题

在新的 Fiber 架构中， 更新时分为两个阶段的 reconciliation 阶段 和 commit 阶段， reconciliation 阶段主要计算更新前后 dom 树的差异，然后是 commit 阶段 把更新渲染到页面上；其中第一个阶段是可以打断的，如果不打断的话，dom 树层级较深 commit 阶段的耗时会过长，并且不能打断，会造成卡死的现象；由于第一阶段会被打断，生命周期会重新执行，会导致在 commit 前的生命周期会执行多次的情况，造成一些意向不到的问题，所以 16 之前的 componentWillMount，componentWillReceiveProps，componetWillUpdate 这些生命周期会被新的生命周期 getDerivedStateFromProps，getSnapshotBeforeUpdate 替换；并且在 17 版本中会删除。

### 参考资料

[正确掌握 React 生命周期 (Lifecycle)](https://juejin.im/entry/587de1b32f301e0057a28897)
