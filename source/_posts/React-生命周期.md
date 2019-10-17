---
title: React 生命周期
date: 2019-10-15 13:59:21
tags:
---

### 16之前旧版生命周期
#### 加载阶段

- constructor() 加载时调用一次，可以在这定义this.state
- getDefaultProps() 设置默认的props ，也可以用dufaultProps设置
- getInitialState() 初始化state
- componentWillMount() 组件加载时调用一次，整个生命周期只调用一次，可以修改state
- render() 创建虚拟dom，diff差异，更新dom
- componentDidMount() 组件渲染之后调用一次

#### 更新阶段

- componentWillreceiveProps(nextProps) 组件接收到新的props调用
- shoundComponentUpate(nextProps, nextState) 组件接收到新的props或 新的state时调用，返回值为bool，false能阻止组件更新
- componentWillUpdate(nextProps, nextState) 只有组件将要更新时调用
- render()
- componentDidUpdate() 组件更新完成之后调用

#### 卸载阶段

- componentWillUnmount() 当组件从 DOM 中移除时会调用

### 16之后新版生命周期
组件的生命周期分为两个阶段，3个运行时期
阶段1： Render 阶段
纯净不包含副作用的，可能被react终止，暂停，或重新启动
阶段2：Commit 阶段
可以使用DOM, 运行副作用，安排更新。
#### 挂载时

- constructor()
- static getDerivedStateFromProps() 在调用 render 方法之前调用，并且在初始挂载及后续更新时都会被调用。它应返回一个对象来更新 state，如果返回 null 则不更新任何内容
- render()
- componentDidMount()

#### 更新时
触发更新render的方式有 new props ， setState()， forceUpdate()

- static getDerivedStateFromProps() 
- shouldComponentUpdate() 根据bool返回值判断组件是否受到props和state更新的影响，首次渲染和forceUpdate时不会调用改方法，应该尽量使用PureComponent, PureComponent 会对 props 和 state 进行浅层比较, 如果数据结构比较复杂是引用类型的话，PureComponent 会出现不被更新的问题，解决方案是使用不可变数据
- render()
- getSnapshotBeforeUpdate() 最近一次渲染输出之前调用，使得能在组件更新之前从dom中捕获一些信息，例如滚动位置；返回值将作为componentDidUpdate()的参数
- componentDidUpdate(prevProps, prevState, snapshot) 组件更新完成调用，首次渲染不会调用，可以在此处对dom进行操作，如果对更新前后的props进行了比较，也可以进行网络请求。

#### 卸载时
- componentWillUnMount() 在组件卸载及销毁之前调用

#### 错误处理
- componentDidCatch() 
- getDerivedStateFromError()

### 新旧版本对比
根据react官方文档的描述，componentWillMount，componentWillreceiveProps，componentWillUpdate 三个生命周期将在未来17版本中移除，所以应该尽量避免在新版本中使用，取而代之的是两个新的生命周期：getDerivedStateFromProps， getSnapshotBeforeUpdate；

componentWillMount 这个生命周期之前一般能做的事情是在这里做一些状态的赋值操作，或者是异步请求，但是状态的赋值可以放在constructor里，在这里异步请求的话也不能第一次render的时候渲染出数据

componentWillreceiveProps 的功能可以使用getDerivedStateFromProps 配合 componentDidUpdate 来完成；使用的场景1：组件有newprops的时候更新state，getDerivedStateFromProps 是一个static方法，所以不能内部使用this; 场景2：当props改变的时候执行一些回调

关于 getDerivedStateFromProps
只有当 state 的值在任何时候都取决于 props的时候才建议使用该生命周期; 否则导致多余的派生状态，这样会导致代码冗余，不好维护。
- 如果要根据props的变更执行一些操作，应该使用componentDidUpdate
- 如果想在props变更的时候重新计算一些数据
- 如果想在props变更的时候重置一些状态，应该使用完全受控组件或有key的非可控组件

需要注意：
- 不要直接复制props的值到state中，应该实现一个受控组件，然后在父组件中合并两个值
- 对于不受控组件，如果想在props变化是重置state；1. 重置所有state，使用key属性 2. 仅更改某些字段的话，观察特殊属性的变化 3. 使用ref调用实例方法

### 为什么组件的生命周期会变动
React16版本引入了异步渲染架构Fiber，由于它主要功能是在组件渲染完成之前，可以中断渲染任务，中断之后组件剩下的生命周期不再执行，而是从头开始执行生命周期

### 为什么需要异步渲染
React16之前对于虚拟dom的渲染是同步的，将每次更改的操作收集起来，和真实dom对比出变化之后，统一一次更新dom树的结构，原来的 stack reconciler 是一个递归执行的函数，父组件调用子组件 reconciler 的过程就是一个递归的过程，如果组件的的层级比较深，dom树比较庞大，这个递归遍历的过程时间会比较长，这个时间段内，浏览器主线程被占用，其它的交互和渲染都会被阻塞，就会出现卡住的感觉

### 怎么实现的异步渲染
React16版本引入了全新的异步渲染架构Fiber，主要对 reconciler 阶段进行了拆分，以后进行详细的探究

### Fiber带来的问题
在新的Fiber架构中， 更新时分为两个阶段的 reconciliation 阶段 和 commit 阶段， reconciliation 阶段主要计算更新前后dom树的差异，然后是commit阶段 把更新渲染到页面上；其中第一个阶段是可以打断的，如果不打断的话，dom树层级较深commit阶段的耗时会过长，并且不能打断，会造成卡死的现象；由于第一阶段会被打断，生命周期会重新执行，会导致在commit前的生命周期会执行多次的情况，造成一些意向不到的问题，所以16之前的componentWillMount，componentWillReceiveProps，componetWillUpdate 这些生命周期会被新的生命周期getDerivedStateFromProps，getSnapshotBeforeUpdate替换；并且在17版本中会删除。