---
title: React事件机制
date: 2020-01-13 11:14:24
tags:
---

## React 事件机制

React 事件机制不是原生的那一套，React 组件上声明的事件没有绑定在 React 组件对应的原生 DOM 节点上，而是绑定在 document 节点上，触发的事件是对原生事件的包装。

```javascript
/**
 * Summary of `ReactBrowserEventEmitter` event handling:
 *
 *  - Top-level delegation is used to trap most native browser events. This
 *    may only occur in the main thread and is the responsibility of
 *    ReactDOMEventListener, which is injected and can therefore support
 *    pluggable event sources. This is the only work that occurs in the main
 *    thread.
 *
 *  - We normalize and de-duplicate events to account for browser quirks. This
 *    may be done in the worker thread.
 *
 *  - Forward these native events (with the associated top-level type used to
 *    trap it) to `EventPluginHub`, which in turn will ask plugins if they want
 *    to extract any synthetic events.
 *
 *  - The `EventPluginHub` will then process each event by annotating them with
 *    "dispatches", a sequence of listeners and IDs that care about that event.
 *
 *  - The `EventPluginHub` then dispatches the events.
 *
 * Overview of React and the event system:
 *
 * +------------+    .
 * |    DOM     |    .
 * +------------+    .
 *       |           .
 *       v           .
 * +------------+    .
 * | ReactEvent |    .
 * |  Listener  |    .
 * +------------+    .                         +-----------+
 *       |           .               +--------+|SimpleEvent|
 *       |           .               |         |Plugin     |
 * +-----|------+    .               v         +-----------+
 * |     |      |    .    +--------------+                    +------------+
 * |     +-----------.--->|EventPluginHub|                    |    Event   |
 * |            |    .    |              |     +-----------+  | Propagators|
 * | ReactEvent |    .    |              |     |TapEvent   |  |------------|
 * |  Emitter   |    .    |              |<---+|Plugin     |  |other plugin|
 * |            |    .    |              |     +-----------+  |  utilities |
 * |     +-----------.--->|              |                    +------------+
 * |     |      |    .    +--------------+
 * +-----|------+    .                ^        +-----------+
 *       |           .                |        |Enter/Leave|
 *       +           .                +-------+|Plugin     |
 * +-------------+   .                         +-----------+
 * | application |   .
 * |-------------|   .
 * |             |   .
 * |             |   .
 * +-------------+   .
 *                   .
 *    React Core     .  General Purpose Event Plugin System
 */
```

React 内部事件系统实现可以分为两个阶段：事件注册、事件触发，涉及到的类有：

- ReactDOMEventListener: 负责事件的注册和分发。将 DOM 时间全都注册到 document 节点上，事件分发主要调用 dispatchEvent 进行，从事件触发组件开始，向父元素遍历。
- ReactBrowserEventEmitter：负责每个 React 组件上事件的执行。
- EventPluginHub： 负责回调函数的存储，注解每个转发过来的事件，然后分发事件。

```javascript
handleClick(e) {
    console.log('click callback', e)
}

render() {
    return (
        <button onClick={this.handleClick}>Click</button>
    )
}
```

用户点击 button 按钮触发 click 事件后，DOM 将 event 传给 ReactDOMEventListener，它将事件分发到当前组件及以上的父组件。然后 ReactBrowserEventEmitter 对每个组件进行事件的执行，先构造 React 合成事件，然后以队列的方式调用 JSX 中声明的 callback。

### 事件注册

从组件的创建和更新来看起，ReactFiberWorkLoop.js -> ReactFiberCompleteWork.js -> ReactDOMHostConfig.js -> ReactDOMComponent.js (setInitialProperties => setInitialDOMProperties => ensureListeningTo ) -> ReactBrowserEventEmitter.js

ReactDOMComponent.js

```javascript
setInitialDOMProperties
else if (registrationNameModules.hasOwnProperty(propKey)) {
    if (nextProp != null) {
        ensureListeningTo(rootContainerElement, propKey);
    }
}

function ensureListeningTo(
  rootContainerElement: Element | Node,
  registrationName: string,
): void {
  const isDocumentOrFragment =
    rootContainerElement.nodeType === DOCUMENT_NODE ||
    rootContainerElement.nodeType === DOCUMENT_FRAGMENT_NODE;
  const doc = isDocumentOrFragment
    ? rootContainerElement
    : rootContainerElement.ownerDocument;
  listenTo(registrationName, doc);
}
```

ReactBrowserEventEmitter.js

```javascript
export function listenTo(
  registrationName: string,
  mountAt: Document | Element | Node
): void {
  const listeningSet = getListenerMapForElement(mountAt);
  const dependencies = registrationNameDependencies[registrationName];

  for (let i = 0; i < dependencies.length; i++) {
    const dependency = dependencies[i];
    listenToTopLevel(dependency, mountAt, listeningSet);
  }
}

export function listenToTopLevel(
  topLevelType: DOMTopLevelEventType,
  mountAt: Document | Element | Node,
  listenerMap: Map<DOMTopLevelEventType | string, null | (any => void)>
): void {
  if (!listenerMap.has(topLevelType)) {
    switch (topLevelType) {
      case TOP_SCROLL:
        trapCapturedEvent(TOP_SCROLL, mountAt);
        break;
      case TOP_FOCUS:
      case TOP_BLUR:
        trapCapturedEvent(TOP_FOCUS, mountAt);
        trapCapturedEvent(TOP_BLUR, mountAt);
        // We set the flag for a single dependency later in this function,
        // but this ensures we mark both as attached rather than just one.
        listenerMap.set(TOP_BLUR, null);
        listenerMap.set(TOP_FOCUS, null);
        break;
      case TOP_CANCEL:
      case TOP_CLOSE:
        if (isEventSupported(getRawEventName(topLevelType))) {
          trapCapturedEvent(topLevelType, mountAt);
        }
        break;
      case TOP_INVALID:
      case TOP_SUBMIT:
      case TOP_RESET:
        // We listen to them on the target DOM elements.
        // Some of them bubble so we don't want them to fire twice.
        break;
      default:
        // By default, listen on the top level to all non-media events.
        // Media events don't bubble so adding the listener wouldn't do anything.
        const isMediaEvent = mediaEventTypes.indexOf(topLevelType) !== -1;
        if (!isMediaEvent) {
          trapBubbledEvent(topLevelType, mountAt);
        }
        break;
    }
    listenerMap.set(topLevelType, null);
  }
}
```

listenToTopLevel 方法解决了不同浏览器间捕获和冒泡不兼容的问题，事件回调方法在冒泡阶段被触发，如果我们想让它在捕获阶段触发，需要在事件名加上 capture，比如冒泡阶段触发，点击事件写成 onClick，在捕获阶段触发需要写成 onCaptureClick。

ReactDOMEventListener.js

```javascript
export function trapBubbledEvent(
  topLevelType: DOMTopLevelEventType,
  element: Document | Element | Node
): void {
  trapEventForPluginEventSystem(element, topLevelType, false);
}

export function trapCapturedEvent(
  topLevelType: DOMTopLevelEventType,
  element: Document | Element | Node
): void {
  trapEventForPluginEventSystem(element, topLevelType, true);
}

function trapEventForPluginEventSystem(
  element: Document | Element | Node,
  topLevelType: DOMTopLevelEventType,
  capture: boolean
): void {
  let listener;
  switch (getEventPriorityForPluginSystem(topLevelType)) {
    case DiscreteEvent:
      listener = dispatchDiscreteEvent.bind(
        null,
        topLevelType,
        PLUGIN_EVENT_SYSTEM
      );
      break;
    case UserBlockingEvent:
      listener = dispatchUserBlockingUpdate.bind(
        null,
        topLevelType,
        PLUGIN_EVENT_SYSTEM
      );
      break;
    case ContinuousEvent:
    default:
      listener = dispatchEvent.bind(null, topLevelType, PLUGIN_EVENT_SYSTEM);
      break;
  }

  const rawEventName = getRawEventName(topLevelType);
  if (capture) {
    addEventCaptureListener(element, rawEventName, listener);
  } else {
    addEventBubbleListener(element, rawEventName, listener);
  }
}
```

EventListener.js

```javascript
export function addEventBubbleListener(
  element: Document | Element | Node,
  eventType: string,
  listener: Function
): void {
  element.addEventListener(eventType, listener, false);
}

export function addEventCaptureListener(
  element: Document | Element | Node,
  eventType: string,
  listener: Function
): void {
  element.addEventListener(eventType, listener, true);
}
```

事件注册就结束了

### 事件分发

React 将所有的直接都委托到了 document 上，那在触发事件的时候，肯定需要一个事件分发的过程，来找到到底是哪个元素触发的事件，并执行相应的事件回调函数，这个将被触发的事件是 React 的合成事件，不是浏览器的原生事件。

在时间注册的阶段，注册到 document 上的事件， 对应的回调函数都会触发 dispatchEvent 方法，在这个方法中

```javascript
// ReactDOMEventListener.js => attemptToDispatchEvent
const nativeEventTarget = getEventTarget(nativeEvent);
let targetInst = getClosestInstanceFromNode(nativeEventTarget);
```

```javascript
function dispatchEventForPluginEventSystem(
  topLevelType: DOMTopLevelEventType,
  eventSystemFlags: EventSystemFlags,
  nativeEvent: AnyNativeEvent,
  targetInst: null | Fiber
): void {
  const bookKeeping = getTopLevelCallbackBookKeeping(
    topLevelType,
    nativeEvent,
    targetInst,
    eventSystemFlags
  );

  try {
    // Event queue being processed in the same cycle allows
    // `preventDefault`.
    batchedEventUpdates(handleTopLevel, bookKeeping);
  } finally {
    releaseTopLevelCallbackBookKeeping(bookKeeping);
  }
}
```

batchedUpdates，就是批处理更新，实际上就是把当前触发的事件放入了批处理队列中，其中，handleTopLevel 是事件分发的核心

```javascript
// ReactDOMEventListener.js
function handleTopLevel(bookKeeping: BookKeepingInstance) {
  let targetInst = bookKeeping.targetInst;

  // Loop through the hierarchy, in case there's any nested components.
  // It's important that we build the array of ancestors before calling any
  // event handlers, because event handlers can modify the DOM, leading to
  // inconsistencies with ReactMount's node cache. See #1105.
  // 事件处理程序，因为事件处理程序可以修改DOM，从而导致与ReactMount的节点缓存不一致。
  let ancestor = targetInst;
  do {
    if (!ancestor) {
      const ancestors = bookKeeping.ancestors;
      ((ancestors: any): Array<Fiber | null>).push(ancestor);
      break;
    }
    const root = findRootContainerNode(ancestor);
    if (!root) {
      break;
    }
    const tag = ancestor.tag;
    if (tag === HostComponent || tag === HostText) {
      bookKeeping.ancestors.push(ancestor);
    }
    ancestor = getClosestInstanceFromNode(root);
  } while (ancestor);
  ...
}
```

首先在事件回调之前，根据当前的组件，向上遍历得到它所有的父组件，存储到 ancestors 中，由于所有的事件都委托到了 document 上，所以在时间触发后，无论是冒泡事件还是捕获事件，它在相关元素上的触发肯定是需要有一定的次序的， 例如在子元素和父元素上都注册了一个鼠标点击冒泡事件，事件触发后，肯定是子元素的事件响应快于父元素，所以在事件队列里，子元素就要排在父元素前面，而在事件回调之前就要进行缓存，原因注释里写到因为事件处理程序可以修改 DOM，从而导致与 ReactMount 的节点缓存不一致。

```javascript
// ReactDOMEventListener.js => handleTopLevel
for (let i = 0; i < bookKeeping.ancestors.length; i++) {
  targetInst = bookKeeping.ancestors[i];
  runExtractedPluginEventsInBatch(
    topLevelType,
    targetInst,
    nativeEvent,
    eventTarget,
    eventSystemFlags
  );
}
```

这里使用了一个 for 循环遍历整个 React 组件和它所有的父组件，执行 runExtractedPluginEventsInBatch 方法，这里的遍历是从前往后遍历，这里分析的是 trapBubbledEvent ，就是冒泡事件，所以这里对应的组件层级上就是由子元素到父元素，runExtractedPluginEventsInBatch 就是事件执行的入口。

### 事件执行

runExtractedPluginEventsInBatch 方法中又调用了两个方法： extractPluginEvents， runEventsInBatch，extractPluginEvents 用于构造合成事件，runEventsInBatch 用于批处理 extractPluginEvents 构造出来的合成事件

#### 构造合成事件

```javascript
// EventPluginHub.js => extractPluginEvents
let events = null;
for (let i = 0; i < plugins.length; i++) {
  // Not every plugin in the ordering may be loaded at runtime.
  const possiblePlugin: PluginModule<AnyNativeEvent> = plugins[i];
  if (possiblePlugin) {
    const extractedEvents = possiblePlugin.extractEvents(
      topLevelType,
      targetInst,
      nativeEvent,
      nativeEventTarget,
      eventSystemFlags
    );
    if (extractedEvents) {
      events = accumulateInto(events, extractedEvents);
    }
  }
}
```

首先遍历 plugins，plugins 是所有合成事件的集合，它们是在 EventPluginHub 初始化阶段注入的

```javascript
// ReactDOMClientInjection.js
EventPluginHubInjection.injectEventPluginsByName({
  SimpleEventPlugin: SimpleEventPlugin,
  EnterLeaveEventPlugin: EnterLeaveEventPlugin,
  ChangeEventPlugin: ChangeEventPlugin,
  SelectEventPlugin: SelectEventPlugin,
  BeforeInputEventPlugin: BeforeInputEventPlugin
});
```

使用了 for 循环把所有的 plugin 执行了一遍，possiblePlugin.extractEvents 调用相应的 plugin 的构造合成事件的方法，例如 SimpleEventPlugin

```javascript
// SimpleEventPlugin.js
const dispatchConfig = topLevelEventsToDispatchConfig.get(topLevelType);
if (!dispatchConfig) {
  return null;
}
```

首先看 topLevelEventsToDispatchConfig 中有没有 topLevelType 属性，有的话说明当前事件可以使用 SimpleEventPlugin 构造合成事件，点击事件的 topLevelType 就是 click， topLevelEventsToDispatchConfig 中的属性是一些常见的事件名称, 然后往下执行

```javascript
// SimpleEventPlugin.js
case DOMTopLevelEventTypes.TOP_CLICK:
    EventConstructor = SyntheticMouseEvent;
    break;
```

SyntheticMouseEvent 可以看成是 SimpleEventPlugin 的一个子 plugin ，SimpleEventPlugin 还有其它的子 plugin， 如 SyntheticWheelEvent，SyntheticClipboardEvent ，SyntheticTouchEvent 等。

设置好 EventConstructor 以后继续执行

```javascript
// SimpleEventPlugin.js
const event = EventConstructor.getPooled(
  dispatchConfig,
  targetInst,
  nativeEvent,
  nativeEventTarget
);
accumulateTwoPhaseDispatches(event);
return event;
```

getPooled 就是从 event 对象池中取出合成事件, React 将所有的事件缓存在对象池中,可以大大降低对象创建和销毁的时间，提升性能

这个 getPooled 其实就是 getPooledEvent，在 SyntheticEvent 初始化的过程中就被设置好初始值了：

```javascript
// SyntheticEvent.js
addEventPoolingTo(SyntheticEvent);

function addEventPoolingTo(EventConstructor) {
  EventConstructor.eventPool = [];
  EventConstructor.getPooled = getPooledEvent;
  EventConstructor.release = releasePooledEvent;
}
```

看下 getPooledEvent 代码

```javascript
// SyntheticEvent.js
function getPooledEvent(dispatchConfig, targetInst, nativeEvent, nativeInst) {
  const EventConstructor = this;
  if (EventConstructor.eventPool.length) {
    const instance = EventConstructor.eventPool.pop();
    EventConstructor.call(
      instance,
      dispatchConfig,
      targetInst,
      nativeEvent,
      nativeInst
    );
    return instance;
  }
  return new EventConstructor(
    dispatchConfig,
    targetInst,
    nativeEvent,
    nativeInst
  );
}
```

首先触发事件的时候需要初始化一下，后续再触发事件的时候，就无需 new 了，直接从对象池中获取。通过 EventConstructor.eventPool.pop() 获取合成对象实例

先看下初始化的流程，执行 new EventConstructor，它可以看成 SyntheticEvent 的子类，就是 SyntheticEvent 扩展得到的，使用 extend 方法。

```javascript
// SyntheticMouseEvent.js
const SyntheticMouseEvent = SyntheticUIEvent.extend({
  screenX: null,
  screenY: null,
  clientX: null,
  clientY: null,
  pageX: null,
  pageY: null,
  ...
});
```

SyntheticMouseEvent 这个合成事件，有自己的一些属性，这些属性其实和浏览器原生的事件回调参数对象 event 的属性没多大差别，都有对于当前事件的一些描述，甚至连属性名都一样，只不过相比于浏览器原生的事件回调参数对象 event 来说，SyntheticMouseEvent 或者说 合成事件 SyntheticEvent 的属性是由 React 主动生成，经过 React 的内部处理，使得其上附加的描述属性完全符合 W3C 的标准，因此在事件层面上具有跨浏览器兼容性，与原生的浏览器事件一样拥有同样的接口，也具备 stopPropagation() 和 preventDefault()等方法

```javascript
handleClick(e) {
    console.log('click callback', e)
}
```

其中 e 其实就是合成事件，而不是浏览器中的原生事件 event，所以开发者无需考虑浏览器兼容性，只需要按照 w3c 规范取值即可；SyntheticUIEvent 主要是往 SyntheticMouseEvent 增加一些属性，SyntheticMouseEvent.extend 又是从 SyntheticEvent 扩展而来的，所以最终会 new SyntheticEvent

```javascript
// SyntheticEvent.js
SyntheticEvent.extend = function(Interface) {
  const Super = this;

  const E = function() {};
  E.prototype = Super.prototype;
  const prototype = new E();

  function Class() {
    return Super.apply(this, arguments);
  }
  Object.assign(prototype, Class.prototype);
  Class.prototype = prototype;
  Class.prototype.constructor = Class;

  Class.Interface = Object.assign({}, Super.Interface, Interface);
  Class.extend = Super.extend;
  addEventPoolingTo(Class);

  return Class;
};
```

React 在这使用了寄生组合继承的方式让 EventConstructor 继承于 SyntheticEvent，使 EventConstructor 获得 SyntheticEvent 上的一些属性和方法，例如 eventPool，getPooled 等，
所以初始化 EventConstructor 的时候，就会调用 SyntheticEvent 的 new 方法，开始构造合成事件，主要就是将原生浏览器事件上的参数挂载到合成事件上，包括 clientX、screenY、timeStamp 等事件属性， preventDefault、stopPropagation 等事件方法

挂载的属性都是经过 React 处理过的，具备跨浏览器能力，同样的，挂载的方法也和原生浏览器的事件方法不同，因为此时的事件是注册在 document 上的，所以调用一些方法，例如 e.stopPropagation() 其实是针对 document 元素调用的，跟原本期望的元素不是同一个，为了让合成事件的表现达到原生事件的的效果，就需要对这些方法进行额外的处理。

```javascript
// SyntheticEvent.js
function functionThatReturnsTrue() {
  return true;
}

stopPropagation: function() {
    const event = this.nativeEvent;
    if (!event) {
      return;
    }

    if (event.stopPropagation) {
      event.stopPropagation();
    } else if (typeof event.cancelBubble !== 'unknown') {
      // The ChangeEventPlugin registers a "propertychange" event for
      // IE. This event does not support bubbling or cancelling, and
      // any references to cancelBubble throw "Member not found".  A
      // typeof check of "unknown" circumvents this issue (and is also
      // IE specific).
      event.cancelBubble = true;
    }

    this.isPropagationStopped = functionThatReturnsTrue;
},
```

首先拿到浏览器原生事件，然后调用对应的 stopPropagation 方法，关键的是 this.isPropagationStopped = functionThatReturnsTrue，在这里设置一个标志，对于冒泡事件来说，当事件触发，由子元素往父元素逐级向上遍历，会按顺序执行每层元素对应的的事件回调，但是如果发现当前元素对应的合成事件上的 isPropagationStopped 为 true 值，则遍历中断，也就不再继续往上遍历，当前元素的所有父元素的合成事件就不会被触发，最终的效果，就和浏览器的原生事件调用 e.stopPropagation 的效果一样的；捕获事件原理一致，只是遍历的顺序相反。

这些事件方法（stopPropagation，preventDefault）一般都是在事件回调函数内调用，而事件的回调函数则是在后面的 accumulateTwoPhaseDispatches 批处理操作中执行的。

```javascript
// SimpleEventPlugin.js
const event = EventConstructor.getPooled(
  dispatchConfig,
  targetInst,
  nativeEvent,
  nativeEventTarget
);
accumulateTwoPhaseDispatches(event);
return event;
```

上面的内容主要是为了初始化得到合成事件， accumulateTwoPhaseDispatches 方法对合成事件进行进一步加工

```javascript
// EventPropagators.js
export function accumulateTwoPhaseDispatches(events) {
  // 遍历所有取到的合成事件执行accumulateTwoPhaseDispatchesSingle方法
  forEachAccumulated(events, accumulateTwoPhaseDispatchesSingle);
}

function accumulateTwoPhaseDispatchesSingle(event) {
  // 检测当前的事件是否具有冒泡阶段或捕获阶段，来决定是否执行 traverseTwoPhase
  if (event && event.dispatchConfig.phasedRegistrationNames) {
    traverseTwoPhase(event._targetInst, accumulateDirectionalDispatches, event);
  }
}

function accumulateDirectionalDispatches(inst, phase, event) {
  // 获取对应的事件回调函数 listener
  const listener = listenerAtPhase(inst, event, phase);
  if (listener) {
    // 将所有相关的listener和 instances 挂载到事件对象的相应属性
    event._dispatchListeners = accumulateInto(
      event._dispatchListeners,
      listener
    );
    event._dispatchInstances = accumulateInto(event._dispatchInstances, inst);
  }
}

function listenerAtPhase(inst, event, propagationPhase: PropagationPhases) {
  const registrationName =
    event.dispatchConfig.phasedRegistrationNames[propagationPhase];
  return getListener(inst, registrationName);
}
```

```javascript
// ReactTreeTraversal.js
// 遍历当前元素以及它所有父元素的集合，按照一定的顺序调用 accumulateDirectionalDispatches
export function traverseTwoPhase(inst, fn, arg) {
  const path = [];
  while (inst) {
    path.push(inst);
    inst = getParent(inst);
  }
  let i;
  for (i = path.length; i-- > 0; ) {
    fn(path[i], "captured", arg);
  }
  for (i = 0; i < path.length; i++) {
    fn(path[i], "bubbled", arg);
  }
}
```

```javascript
// accumulateInto.js accumulateInto
// listenerAtPhase 执行后返回 getListener 得到的事件回调函数(next) ,按照顺序与当前已经存储好的事件回调合并
if (current == null) {
  return next;
}

// Both are not empty. Warning: Never call x.concat(y) when you are not
// certain that x is an Array (x could be a string with concat method).
if (Array.isArray(current)) {
  if (Array.isArray(next)) {
    current.push.apply(current, next);
    return current;
  }
  current.push(next);
  return current;
}

if (Array.isArray(next)) {
  // A bit too dangerous to mutate `next`.
  return [current].concat(next);
}

return [current, next];
```

上面的加工过程大致步骤就是保存当前元素及它父元素上的挂载的所有事件回调函数，包括冒泡事件和捕获事件，保存到 event 的\_dispatchListeners 属性上，并且将当前元素及它父元素的 react 实例保存在 event 的\_dispatchInstances 属性上。接下来就可以对合成事件进行批量处理了

### 批处理合成事件

runEventsInBatch 是入口

```javascript
// EventBatching.js runEventsInBatch

let eventQueue: ?(Array<ReactSyntheticEvent> | ReactSyntheticEvent) = null;
if (events !== null) {
  eventQueue = accumulateInto(eventQueue, events);
}

// Set `eventQueue` to null before processing it so that we can tell if more
// events get enqueued while processing.
const processingEventQueue = eventQueue;
eventQueue = null;

if (!processingEventQueue) {
  return;
}

forEachAccumulated(processingEventQueue, executeDispatchesAndReleaseTopLevel);
// This would be a good time to rethrow if any of the event handlers threw.
rethrowCaughtError();
```

这个方法首先会将当前需要处理的 events 事件，与之前没有处理完毕的队列调用 accumulateInto 方法按照顺序进行合并，组合成一个新的队列，因为之前可能就存在还没处理完的合成事件，这里就又有得到执行的机会了

```javascript
// forEachAccumulated.js
function forEachAccumulated<T>(
  arr: ?(Array<T> | T),
  cb: (elem: T) => void,
  scope: ?any
) {
  if (Array.isArray(arr)) {
    arr.forEach(cb, scope);
  } else if (arr) {
    cb.call(scope, arr);
  }
}
```

forEachAccumulated 方法先看下事件队列 processingEventQueue 是不是个数组，如果是数组，说明队列中不止一个事件，则遍历队列，调用 executeDispatchesAndReleaseTopLevel 方法，否则说明队列中只有一个事件，则无需遍历直接调用。

```javascript
// EventBatching.js
const executeDispatchesAndRelease = function(event: ReactSyntheticEvent) {
  if (event) {
    executeDispatchesInOrder(event);

    if (!event.isPersistent()) {
      event.constructor.release(event);
    }
  }
};
const executeDispatchesAndReleaseTopLevel = function(e) {
  return executeDispatchesAndRelease(e);
};
```

```javascript
export function executeDispatchesInOrder(event) {
  const dispatchListeners = event._dispatchListeners;
  const dispatchInstances = event._dispatchInstances;
  if (Array.isArray(dispatchListeners)) {
    for (let i = 0; i < dispatchListeners.length; i++) {
      if (event.isPropagationStopped()) {
        break;
      }
      // Listeners and Instances are two parallel arrays that are always in sync.
      executeDispatch(event, dispatchListeners[i], dispatchInstances[i]);
    }
  } else if (dispatchListeners) {
    executeDispatch(event, dispatchListeners, dispatchInstances);
  }
  event._dispatchListeners = null;
  event._dispatchInstances = null;
}

export function executeDispatch(event, listener, inst) {
  const type = event.type || "unknown-event";
  event.currentTarget = getNodeFromInstance(inst);
  invokeGuardedCallbackAndCatchFirstError(type, listener, undefined, event);
  event.currentTarget = null;
}
```

首先对拿到的事件上挂在的 dispatchListeners，也就是之前拿到的当前元素以及其所有父元素上注册的事件回调函数的集合，遍历这个集合，如果发现遍历到的事件的 event.isPropagationStopped()为 true，则遍历的循环直接 break 掉，这里的 isPropagationStopped ，它是用于标识当前 React Node 上触发的事件是否执行了 e.stopPropagation()这个方法，如果执行了，则说明在此之前触发的事件已经调用 event.stopPropagation()，isPropagationStopped 的值被置为 functionThatReturnsTrue，即执行后为 true，当前事件以及后面的事件作为父级事件就不应该再被执行了

这里当 event.isPropagationStopped()为 true 时，中断合成事件的向上遍历执行，也就起到了和原生事件调用 stopPropagation 相同的效果

```javascript
// ReactErrorUtils.js
export function invokeGuardedCallbackAndCatchFirstError<
  A,
  B,
  C,
  D,
  E,
  F,
  Context
>(
  name: string | null,
  func: (a: A, b: B, c: C, d: D, e: E, f: F) => void,
  context: Context,
  a: A,
  b: B,
  c: C,
  d: D,
  e: E,
  f: F
): void {
  invokeGuardedCallback.apply(this, arguments);
  if (hasError) {
    const error = clearCaughtError();
    if (!hasRethrowError) {
      hasRethrowError = true;
      rethrowError = error;
    }
  }
}

export function invokeGuardedCallback<A, B, C, D, E, F, Context>(
  name: string | null,
  func: (a: A, b: B, c: C, d: D, e: E, f: F) => mixed,
  context: Context,
  a: A,
  b: B,
  c: C,
  d: D,
  e: E,
  f: F
): void {
  hasError = false;
  caughtError = null;
  invokeGuardedCallbackImpl.apply(reporter, arguments);
}

// invokeGuardedCallbackImpl.js

let invokeGuardedCallbackImpl = function<A, B, C, D, E, F, Context>(
  name: string | null,
  func: (a: A, b: B, c: C, d: D, e: E, f: F) => mixed,
  context: Context,
  a: A,
  b: B,
  c: C,
  d: D,
  e: E,
  f: F
) {
  const funcArgs = Array.prototype.slice.call(arguments, 3);
  try {
    func.apply(context, funcArgs);
  } catch (error) {
    this.onError(error);
  }
};
```

关键是 `func.apply(context, funcArgs)` func 就是传入的事件回调， funcArgs 就是合成事件对象，所以我们能在事件回调参数上拿到合成事件对象，到这里事件执行就完了

### 事件的清理

事件执行完毕以后，接下来就需要清理工作了，因为 React 采用了对象池的方式来管理合成事件，所以当事件执行完毕以后就需要清理释放掉，减少内存的占用，只要是执行了 executeDispatchesAndRelease 方法中的 `event.constructor.release(event);`

```javascript
// SyntheticEvent.js
function releasePooledEvent(event) {
  const EventConstructor = this;
  event.destructor();
  if (EventConstructor.eventPool.length < EVENT_POOL_SIZE) {
    EventConstructor.eventPool.push(event);
  }
}
```

这个方法主要做了两件事，首先释放掉 event 上属性占用的内存，然后把清理后的 event 对象再放入对象池中，可以被后续事件对象二次利用 event.destructor 是 SyntheticEvent 上的方法，用来释放内存的

```javascript
destructor: function() {
    const Interface = this.constructor.Interface;
    for (const propName in Interface) {
      if (__DEV__) {
        Object.defineProperty(
          this,
          propName,
          getPooledWarningPropertyDefinition(propName, Interface[propName]),
        );
      } else {
        this[propName] = null;
      }
    }
    this.dispatchConfig = null;
    this._targetInst = null;
    this.nativeEvent = null;
    this.isDefaultPrevented = functionThatReturnsFalse;
    this.isPropagationStopped = functionThatReturnsFalse;
    this._dispatchListeners = null;
    this._dispatchInstances = null;
  },
```

### 总结

React 的事件机制主要使用事件委托的原理，对浏览器的事件进行优化消除了浏览器间的差异，主要分为 事件注册，事件分发，事件执行的时候主要使用 EventPluginHub 来 负责调度事件的存储，合成事件和对对象池的方式实现创建和销毁。
