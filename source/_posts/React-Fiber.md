---
title: React Fiber
date: 2019-10-31 17:15:06
tags:
---

### Fiber 出现的原因

Fiber 出现的使命主要解决大型 React 项目的性能问题，主要出现在大量渲染 dom 的场景：主要原因是浏览器是单线程的，它的 GUI 渲染，定时器处理，事件处理，js 执行，资源加载都放在一起去，当做某件事，只有将它昨晚才能做下一件事情，如果时间足够的话，浏览器会对代码进行优化和 reflow 处理，所以大量的 dom 渲染会造成渲染时间过长，导致性能问题，浏览器渲染不能中断，但是 js 的执行一些是可以控制的，让她们分批执行，task 的时长不能太长，这样浏览器就能优化代码和 reflow 处理，性能就能有提升；所以说 Fiber 是一个异步渲染的架构，利用时间分片把大量的 dom 渲染任务拆分开来，小批量执行，从而提高性能。

#### Fiber 基础

 递归遍历

Jsx 是 javaScript 语法的一个扩展，在 React 里可以很好的描述 UI 和组件， 标签是天然的嵌套结构，树形结构，要遍历树形结构最简单直观的方法是递归遍历；就是编译之后会成为递归执行的代码，所以 React16 之前的调度器被称为栈调度器，但是栈的问题是不能随意的断开和继续，所以不能满足需求的，所以需要使用新的结构，链表的结构对于异步是友好的，链表在循环的时候不需要每次都进入递归栈

Filber 结构-链表遍历

要实现该遍历算法，需要一个包含 3 个字段的数据结构

- child 第一个子节点的引用
- sibling 第一个兄弟节点的引用
- return 父节点的引用

下面的遍历算法是父节点优先，深度优先的实现。大概思路是保持对当前节点的引用，并在向下遍历树时重新给它赋值，直到我们到达分支的末尾。然后我们使用 return 指针返回根节点。

```javascript
let root = fiber;
let node = fiber;
while (true) {
  // Do something with node
  if (node.child) {
    node = node.child;
    continue;
  }
  if (node === root) {
    return;
  }
  while (!node.sibling) {
    if (!node.return || node.return === root) {
      return;
    }
    node = node.return;
  }
  node = node.sibling;
}
le;
```

Demo: [地址](https://codesandbox.io/s/amazing-goldberg-wv5jf)

React 中的的工作循环

ReactFiberWorkLoop.js

```javascript
// The work loop is an extremely hot path. Tell Closure not to inline it.
function workLoopSync() {
  // Already timed out, so perform work without checking if we need to yield.
  while (workInProgress !== null) {
    workInProgress = performUnitOfWork(workInProgress);
  }
}

function workLoopConcurrent() {
  // Perform work until Scheduler asks us to yield
  while (workInProgress !== null && !shouldYield()) {
    workInProgress = performUnitOfWork(workInProgress);
  }
}
```

workInProgress 变量作为顶部帧，保留对当前节点的引用，这里的算法可以同步的遍历组件树，并且为树中的每个 Fiber 节点执行工作。一般是交互事件引起的 UI 变更更新。或者是异步的遍历组件树，检查在执行 Fiber 节点工作之后是否还剩下时间，shouldYield 函数返回基于 expirationTime 和 deadline 变量的结果，这些变量会在 Fiber 节点执行工作的时候更新。

Fiber 的架构主要分为两个主要阶段： 协调/渲染和提交；在源码中协调阶段通常称为渲染阶段，是 React 遍历组件树的阶段

- 更新状态和熟悉
- 调用生命周期钩子
- 获取组件的 `children`
- 将它们和之前的 `children` 对比
- 并且计算出需要执行的 dom 更新

这些是 Fiber 内部的工作，工作的类型由 React Element 的类型，比如 `ClassComponent` 需要实例化一个类，而 `FunctionComponent` 却不需要。

Fiber 工作目标类型有

```javascript
export const FunctionComponent = 0;
export const ClassComponent = 1;
export const IndeterminateComponent = 2; // Before we know whether it is function or class
export const HostRoot = 3; // Root of a host tree. Could be nested inside another node.
export const HostPortal = 4; // A subtree. Could be an entry point to a different renderer.
export const HostComponent = 5;
export const HostText = 6;
export const Fragment = 7;
export const Mode = 8;
export const ContextConsumer = 9;
export const ContextProvider = 10;
export const ForwardRef = 11;
export const Profiler = 12;
export const SuspenseComponent = 13;
export const MemoComponent = 14;
export const SimpleMemoComponent = 15;
export const LazyComponent = 16;
export const IncompleteClassComponent = 17;
export const DehydratedFragment = 18;
export const SuspenseListComponent = 19;
export const FundamentalComponent = 20;
export const ScopeComponent = 21;
```

### React 调度原理

因为浏览器是在一个线程中处理 js 代码执行，页面渲染的，所以 js 代码在执行的时候渲染引擎是停止工作的，如果是一个复杂组件需要重新渲染，js 的调用栈会很长，如果还有些耗时的复制操作，就可能导致长时间阻塞渲染引擎，带啦不好的用户体验，调度就是要解决这个问题而出现的。

并发(Concurrent) React, 也称为时间分片(Time slicing)。在 16 版本新的架构之后，React 可以允许渲染过程分段完成，中间可以返回主线程执行其他任务。

#### 调度器的实现

使用间隔较短的 postMessage 循环，而不是尝试使用 requestAnimationFrame 对齐框架边界。

```javascript
const channel = new MessageChannel();
const port = channel.port2;
channel.port1.onmessage = performWorkUntilDeadline;
const performWorkUntilDeadline = () => {
  if (scheduledHostCallback !== null) {
    const currentTime = getCurrentTime();
    // Yield after `yieldInterval` ms, regardless of where we are in the vsync
    // cycle. This means there's always time remaining at the beginning of
    // the message event.
    deadline = currentTime + yieldInterval;
    const hasTimeRemaining = true;
    try {
      const hasMoreWork = scheduledHostCallback(hasTimeRemaining, currentTime);
      if (!hasMoreWork) {
        isMessageLoopRunning = false;
        scheduledHostCallback = null;
      } else {
        // If there's more work, schedule the next message event at the end
        // of the preceding one.
        port.postMessage(null);
      }
    } catch (error) {
      // If a scheduler task throws, exit the current browser task so the
      // error can be observed.
      port.postMessage(null);
      throw error;
    }
  } else {
    isMessageLoopRunning = false;
  }
  // Yielding to the browser will give it a chance to paint, so we can
  // reset this.
  needsPaint = false;
};
```

React 会根据任务的优先级去分配各自的 `expirationTime` , 在过期时间到之前先去处理高优先级的任务，并且高优先级的任务可以打断低优先级的任务，所以会导致一些生命周期函数多次执行的问题。

优先级类型

```javascript
export const NoPriority = 0;
export const ImmediatePriority = 1; // 立即执行优先级，同步执行
export const UserBlockingPriority = 2; // 用户阻塞优先级 250ms 过期 需要用户交互结果允许的任务，点击
export const NormalPriority = 3; // 普通优先级 5ms 过期 不必让用户立即感受到的更新
export const LowPriority = 4; // 低优先级 10ms 过期 可以推迟但是最终需要完成的任务
export const IdlePriority = 5; // 空闲优先级 永不过期
```

#### 调度的流程

Scheduler.js 调度器的入口

```javascript
function unstable_scheduleCallback(priorityLevel, callback, options) {
  var currentTime = getCurrentTime();

  var startTime;
  var timeout;
  if (typeof options === "object" && options !== null) {
    var delay = options.delay;
    if (typeof delay === "number" && delay > 0) {
      startTime = currentTime + delay;
    } else {
      startTime = currentTime;
    }
    timeout =
      typeof options.timeout === "number"
        ? options.timeout
        : timeoutForPriorityLevel(priorityLevel);
  } else {
    timeout = timeoutForPriorityLevel(priorityLevel);
    startTime = currentTime;
  }

  var expirationTime = startTime + timeout;

  var newTask = {
    id: taskIdCounter++,
    callback,
    priorityLevel,
    startTime,
    expirationTime,
    sortIndex: -1
  };

  if (startTime > currentTime) {
    // This is a delayed task.
    newTask.sortIndex = startTime;
    push(timerQueue, newTask);
    if (peek(taskQueue) === null && newTask === peek(timerQueue)) {
      // All tasks are delayed, and this is the task with the earliest delay.
      if (isHostTimeoutScheduled) {
        // Cancel an existing timeout.
        cancelHostTimeout();
      } else {
        isHostTimeoutScheduled = true;
      }
      // Schedule a timeout.
      requestHostTimeout(handleTimeout, startTime - currentTime);
    }
  } else {
    newTask.sortIndex = expirationTime;
    push(taskQueue, newTask);
    // Schedule a host callback, if needed. If we're already performing work,
    // wait until the next time we yield.
    if (!isHostCallbackScheduled && !isPerformingWork) {
      isHostCallbackScheduled = true;
      requestHostCallback(flushWork);
    }
  }

  return newTask;
}
```
