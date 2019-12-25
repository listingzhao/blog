---
title: ReactHooks的工作原理
date: 2019-11-20 17:10:45
tags:
---

#### Dispatcher

Dispatcher 是包含了 hooks 函数的共享对象，它将根据 ReactDom 的渲染阶段来动态分配和清除，并且确保用户无法在 React 组件外访问 hooks

当我们执行完渲染工作时，我们将 Dispatcher 置空从而防止它在 ReactDOM 的渲染周期之外被意外的调用。
ReactFiberWorkLoop.js
[具体实现](https://github.com/facebook/react/blob/master/packages/react-reconciler/src/ReactFiberWorkLoop.js#L1494)

Dispatcher 在每一次 hook 中都是用 resolveDispatcher()函数调用。[具体实现](https://github.com/facebook/react/blob/master/packages/react/src/ReactHooks.js#L22)

Dispatcher 是 hooks 一个简单的封装机制

#### Hooks Queue

ReactPartialRendererHooks.js

mountState 源码

```javascript
function mountWorkInProgressHook() {
  const hook = {
    memoizedState: null,

    baseState: null,
    queue: null,
    baseUpdate: null,

    next: null
  };

  if (workInProgressHook === null) {
    // This is the first hook in the list
    firstWorkInProgressHook = workInProgressHook = hook;
  } else {
    // Append to the end of the list
    workInProgressHook = workInProgressHook.next = hook;
  }
  return workInProgressHook;
}

function mountState(initialState) {
  const hook = mountWorkInProgressHook();
  if (typeof initialState === "function") {
    initialState = initialState();
  }
  hook.memoizedState = hook.baseState = initialState;
  const queue = (hook.queue = {
    last: null,
    dispatch: null,
    lastRenderedReducer: basicStateReducer,
    lastRenderedState: initialState
  });
  const dispatch = (queue.dispatch = dispatchAction.bind(
    null,
    // Flow doesn't know this is non-null, but we do.
    currentlyRenderingFiber,
    queue
  ));
  return [hook.memoizedState, dispatch];
}
```

dispatchAction 源码

```javascript
```
