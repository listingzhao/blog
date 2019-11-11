---
title: React Fiber
date: 2019-10-31 17:15:06
tags:
---

### Fiber 出现的原因

Fiber 出现的使命主要解决大型 React 项目的性能问题，主要出现在大量渲染 dom 的场景：主要原因是浏览器是单线程的，它的 GUI 渲染，定时器处理，事件处理，js 执行，资源加载都放在一起去，当做某件事，只有将它昨晚才能做下一件事情，如果时间足够的话，浏览器会对代码进行优化和 reflow 处理，所以大量的 dom 渲染会造成渲染时间过长，导致性能问题，浏览器渲染不能中断，但是 js 的执行一些是可以控制的，让她们分批执行，task 的时长不能太长，这样浏览器就能优化代码和 reflow 处理，性能就能有提升；所以说 Fiber 是一个异步渲染的架构，利用时间分片把大量的 dom 渲染任务拆分开来，小批量执行，从而提高性能。

#### Fiber 基础

Jsx 是 javaScript 语法的一个扩展，在 React 里可以很好的描述 UI 和组件， 标签是天然的嵌套结构；就是编译之后会成为递归执行的代码，所以 React16 之前的调度器被称为栈调度器，但是栈的问题是不能随意的断开和继续，所以不能满足需求的，所以需要使用新的结构，链表的结构对于异步是友好的，链表  在循环的时候不需要每次都进入递归栈

Fiber 的架构主要分为两个主要阶段： 协调/渲染和提交；在源码中协调阶段通常称为渲染阶段，是 React 遍历组件树的阶段

-   更新状态和熟悉
-   调用生命周期钩子
-   获取组件的 `children`
-   将它们和之前的 `children` 对比
-   并且计算出需要执行的 dom 更新

这些是 Fiber 内部的工作，工作的类型由 React Element 的类型，比如 `ClassComponent` 需要实例化一个类，而 `FunctionComponent` 却不需要。

Fiber 工作目标类型有

```javascript
export const FunctionComponent = 0
export const ClassComponent = 1
export const IndeterminateComponent = 2 // Before we know whether it is function or class
export const HostRoot = 3 // Root of a host tree. Could be nested inside another node.
export const HostPortal = 4 // A subtree. Could be an entry point to a different renderer.
export const HostComponent = 5
export const HostText = 6
export const Fragment = 7
export const Mode = 8
export const ContextConsumer = 9
export const ContextProvider = 10
export const ForwardRef = 11
export const Profiler = 12
export const SuspenseComponent = 13
export const MemoComponent = 14
export const SimpleMemoComponent = 15
export const LazyComponent = 16
export const IncompleteClassComponent = 17
export const DehydratedFragment = 18
export const SuspenseListComponent = 19
export const FundamentalComponent = 20
export const ScopeComponent = 21
```

### React 调度原理
