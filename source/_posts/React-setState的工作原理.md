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

### setState forceUpdate

setState 是在 React 基础模块中定义的，相关实现是在渲染器中。

ReactBaseClasses.js

```javascript
Component.prototype.setState = function(partialState, callback) {
  this.updater.enqueueSetState(this, partialState, callback, "setState");
};

Component.prototype.forceUpdate = function(callback) {
  this.updater.enqueueForceUpdate(this, callback, "forceUpdate");
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
  },
  enqueueForceUpdate(inst, callback) {
    const fiber = getInstance(inst);
    const currentTime = requestCurrentTimeForUpdate();
    const suspenseConfig = requestCurrentSuspenseConfig();
    const expirationTime = computeExpirationForFiber(
      currentTime,
      fiber,
      suspenseConfig
    );

    const update = createUpdate(expirationTime, suspenseConfig);
    // 与setState不同的地方 默认是0  ForceUpdate = 2
    update.tag = ForceUpdate;

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

export function createUpdateQueue(baseState) {
  const queue = {
    baseState,
    firstUpdate: null,
    lastUpdate: null,
    firstCapturedUpdate: null,
    lastCapturedUpdate: null,
    firstEffect: null,
    lastEffect: null,
    firstCapturedEffect: null,
    lastCapturedEffect: null
  };
  return queue;
}

export function enqueueUpdate(fiber, update) {
  const alternate = fiber.alternate;
  let queue1;
  let queue2;
  if (alternate === null) {
    // There's only one fiber.
    queue1 = fiber.updateQueue;
    queue2 = null;
    if (queue1 === null) {
      queue1 = fiber.updateQueue = createUpdateQueue(fiber.memoizedState);
    }
  }
  if (queue2 === null || queue1 === queue2) {
    // There's only a single queue.
    appendUpdateToQueue(queue1, update);
  }
}

function appendUpdateToQueue(queue, update) {
  // Append the update to the end of the list.
  if (queue.lastUpdate === null) {
    // Queue is empty
    queue.firstUpdate = queue.lastUpdate = update;
  } else {
    queue.lastUpdate.next = update;
    queue.lastUpdate = update;
  }
}
```

ReactFiberWorkLoop.js

```javascript
export const scheduleWork = scheduleUpdateOnFiber;
export function scheduleUpdateOnFiber(fiber, expirationTime) {
  if (expirationTime === Sync) {
    if (
      // Check if we're inside unbatchedUpdates
      (executionContext & LegacyUnbatchedContext) !== NoContext &&
      // Check if we're not already rendering
      (executionContext & (RenderContext | CommitContext)) === NoContext
    ) {
      // Register pending interactions on the root to avoid losing traced interaction data.
      schedulePendingInteractions(root, expirationTime);

      // This is a legacy edge case. The initial mount of a ReactDOM.render-ed
      // root inside of batchedUpdates should be synchronous, but layout updates
      // should be deferred until the end of the batch.
      performSyncWorkOnRoot(root);
    }
  }
}

function performSyncWorkOnRoot(root) {
  // Check if there's expired work on this root. Otherwise, render at Sync.
  const lastExpiredTime = root.lastExpiredTime;
  const expirationTime = lastExpiredTime !== NoWork ? lastExpiredTime : Sync;
  if (root.finishedExpirationTime === expirationTime) {
    // There's already a pending commit at this expiration time.
    // TODO: This is poorly factored. This case only exists for the
    // batch.commit() API.
    commitRoot(root);
  } else {
    // Before exiting, make sure there's a callback scheduled for the next
    // pending level.
    ensureRootIsScheduled(root);
  }
}

function ensureRootIsScheduled(root) {
  const lastExpiredTime = root.lastExpiredTime;
  if (lastExpiredTime !== NoWork) {
    // Special case: Expired work should flush synchronously.
    root.callbackExpirationTime = Sync;
    root.callbackPriority = ImmediatePriority;
    root.callbackNode = scheduleSyncCallback(
      performSyncWorkOnRoot.bind(null, root)
    );
    return;
  }
}
```

SchedulerWithReactIntegration.js

```javascript
export function scheduleSyncCallback(callback: SchedulerCallback) {
  // Push this callback into an internal queue. We'll flush these either in
  // the next tick, or earlier if something calls `flushSyncCallbackQueue`.
  if (syncQueue === null) {
    syncQueue = [callback];
    // Flush the queue in the next tick, at the earliest.
    immediateQueueCallbackNode = Scheduler_scheduleCallback(
      Scheduler_ImmediatePriority,
      flushSyncCallbackQueueImpl
    );
  } else {
    // Push onto existing queue. Don't need to schedule a callback because
    // we already scheduled one when we created the queue.
    syncQueue.push(callback);
  }
  return fakeCallbackNode;
}
```

Scheduler_scheduleCallback 为 Scheduler 调度器的入口 unstable_scheduleCallback，Scheduler 的具体内容见 Fiber 相关。

#### 总结

- 根据 currentTime 和 fiber 节点对象计算出 expirationTime
- 根据 expirationTime 创建 update 对象
- 将 setState 中要更新的对象 赋值给 update.payload
- 将 setState 中的 callback 赋值给 update.callback
- 将 update 加入更新队列 updateQueue
- 进行任务调度
