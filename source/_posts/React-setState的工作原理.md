---
title: React setState的工作原理
date: 2019-11-01 14:04:50
tags:
---

React 源码部分有三个核心部分，主要分为 react 基础包、renderer 渲染模块、react-reconciler 核心模块

### react 基础模块

主要定义了 React 的基本 api 和组件类。

```javascript
const React = {
  Children: {
    map,
    forEach,
    count,
    toArray,
    only,
  },

  createRef,
  Component,
  PureComponent,

  createContext,
  forwardRef,
  lazy,
  memo,

  useCallback,
  useContext,
  useEffect,
  useImperativeHandle,
  useDebugValue,
  useLayoutEffect,
  useMemo,
  useReducer,
  useRef,
  useState,

  createElement,
  cloneElement,
  createFactory,
  isValidElement,
  ...
};
```

### renderer 渲染模块

renderer 模块的作用主要是用来更新视图的，根据不同的环境可能会有不同的渲染模块，例如我们熟悉的 react-dom，还有其它的 react-native-renderer 等;依赖 react-reconciler 模块，注入相应的渲染方法到 reconciler 中。react-dom 的基本 api：

```javascript
const ReactDOM: Object = {
  createPortal,
  findDOMNode(
    componentOrElement: Element | ?React$Component<any, any>
  ): null | Element | Text {},
  hydrate(element: React$Node, container: DOMContainer, callback: ?Function) {},
  render(
    element: React$Element<any>,
    container: DOMContainer,
    callback: ?Function
  ) {},
  unmountComponentAtNode(container: DOMContainer) {},
  flushSync: flushSync
};
```

### react-reconciler 核心模块

主要负责调度算法和 Fiber tree diff，连接 React 基础模块和 rederer 渲染模块。

### setState

setState 是在 React 基础模块中定义的，相关实现是在渲染器中。

ReactBaseClasses.js

```javascript
Component.prototype.setState = function(partialState, callback) {
  this.updater.enqueueSetState(this, partialState, callback, "setState");
};
```

ReactFiberClassComponent.js

```javascript
const classComponentUpdater = {
  isMounted,
  enqueueSetState(inst, payload, callback) {
    const fiber = getInstance(inst);
    const currentTime = requestCurrentTimeForUpdate();
    const suspenseConfig = requestCurrentSuspenseConfig();
    const expirationTime = computeExpirationForFiber(
      currentTime,
      fiber,
      suspenseConfig
    );

    const update = createUpdate(expirationTime, suspenseConfig);
    update.payload = payload;
    if (callback !== undefined && callback !== null) {
      update.callback = callback;
    }

    enqueueUpdate(fiber, update);
    scheduleWork(fiber, expirationTime);
  }
};
```

ReactUpdateQueue.js

```javascript
export const UpdateState = 0; // 更新
export const ReplaceState = 1; // 替换
export const ForceUpdate = 2; // 强制更新
export const CaptureUpdate = 3; // 捕获性的更新

export function createUpdate(expirationTime, suspenseConfig) {
  let update = {
    expirationTime,
    suspenseConfig,

    tag: UpdateState,
    payload: null,
    callback: null,
    // 指向下一个更新
    next: null,
    // 指向下一个副作用
    nextEffect: null
  };
  return update;
}
```
