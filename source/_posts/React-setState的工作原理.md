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
    next: null
  };

  update.next = update;
  return update;
}

export function enqueueUpdate(fiber, update) {
  const updateQueue = fiber.updateQueue;
  if (updateQueue === null) {
    // Only occurs if the fiber has been unmounted.
    return;
  }

  const sharedQueue = updateQueue.shared;
  const pending = sharedQueue.pending;
  if (pending === null) {
    // 首次更新创建一个循环链表
    update.next = update;
  } else {
    update.next = pending.next;
    pending.next = update;
  }
  sharedQueue.pending = update;
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
  // If we have a work-in-progress fiber, it means there's still work to do
  // in this root.
  if (workInProgress !== null) {
    const prevExecutionContext = executionContext;
    executionContext |= RenderContext;
    const prevDispatcher = pushDispatcher(root);
    const prevInteractions = pushInteractions(root);
    startWorkLoopTimer(workInProgress);

    do {
      try {
        // fiber 工作循环
        workLoopSync();
        break;
      } catch (thrownValue) {
        handleError(root, thrownValue);
      }
    } while (true);
    resetContextDependencies();
    executionContext = prevExecutionContext;
    popDispatcher(prevDispatcher);
    if (enableSchedulerTracing) {
      popInteractions(((prevInteractions: any): Set<Interaction>));
    }

    if (workInProgressRootExitStatus === RootFatalErrored) {
      const fatalError = workInProgressRootFatalError;
      stopInterruptedWorkLoopTimer();
      prepareFreshStack(root, expirationTime);
      markRootSuspendedAtTime(root, expirationTime);
      ensureRootIsScheduled(root);
      throw fatalError;
    }

    if (workInProgress !== null) {
      // This is a sync render, so we should have finished the whole tree.
      invariant(
        false,
        "Cannot commit an incomplete root. This error is likely caused by a " +
          "bug in React. Please file an issue."
      );
    } else {
      // We now have a consistent tree. Because this is a sync render, we
      // will commit it even if something suspended.
      stopFinishedWorkLoopTimer();
      root.finishedWork = (root.current.alternate: any);
      root.finishedExpirationTime = expirationTime;
      finishSyncRender(root, workInProgressRootExitStatus, expirationTime);
    }

    // Before exiting, make sure there's a callback scheduled for the next
    // pending level.
    ensureRootIsScheduled(root);
  }
  return null;
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
