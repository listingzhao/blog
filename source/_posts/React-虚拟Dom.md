---
title: React 虚拟DOM
date: 2019-12-19 11:10:02
tags:
---

### 虚拟 DOM

在传统的 JavaScript 程序中，我们会通过 JavaScript 直接对 dom 进行操作，dom 元素通过我们监听的事件来和应用程序通讯，例如 jQuery 就是一个传统的比较方便 dom 操作类库，提供了一些非常便利的操作 dom 的方法。

而在 React 中，会将真实的 dom 结构抽象为 JavaScript 对象，React 会先将 JSX 代码转换为这个 JavaScript 对象，然后再将这个 JavaScript 对象更新为真实的 dom 结构；这个 JavaScript 对象就称为虚拟 DOM。

### 使用虚拟 DOM 的原因

#### 提升 dom 更新的性能

React 将 Dom 抽象为虚拟 DOM，然后通过 diff 算法比较新旧虚拟 Dom 两个对象的差异，最终只把变化的部分重新渲染，并且支持批处理，不是立马更新，大幅减少 DOM 操作带来的重计算步骤，提高了 Dom 更新渲染的效率。

#### 提升开发效率

使用传统的 javaScript 程序中，开发者更多的可能会关注如何去操作更新 dom。使用 React 以后，开发者只需要告诉 React 当前视图处于什么状态，它会通过虚拟 Dom 确保真实 dom 与该状态匹配。开发者不需要去关心如果操作更新 Dom，以及事件的处理。开发者可以更关注与业务逻辑而不是 Dom 操作，这一点可以大大提升我们的开发效率。

#### 解决浏览器兼容

React 基于虚拟 Dom 实现一套标准 API，Dom 操作，事件处理，抹平了各个浏览器的事件兼容性问题。

#### 解决平台兼容

React 基于虚拟 Dom 可以实现不同的渲染层，例如 React Native。

### 虚拟 DOM 实现原理

#### JSX 和 createElement

创建一个 React 的 UI 组件可以使用 JSX，也可以使用 createElement。JSX 需要配合 Babel 使用，最终 JSX 会被 Babel 编译成 createElement 的形式，JSX 就是一种 createElement 的语法糖。

```html
<div>
  <h2>hello</h2>
</div>
```

```javascript
// jsx
class Word extends React.Component {
  render() {
    return <div>word</div>;
  }
}
class Hello extends React.Component {
  render() {
    return (
      <div>
        <h2>hello</h2>
        <Word />
      </div>
    );
  }
}

// createElement
class Word extends React.Component {
  render() {
    return React.createElement("div", null, "word");
  }
}

class Hello extends React.Component {
  render() {
    return React.createElement(
      "div",
      null,
      React.createElement("h2", null, "hello", React.createElement(Word, null))
    );
  }
}
```

#### 创建虚拟 Dom

```javascript
export function createElement(type, config, children) {
  let propName;

  const props = {};

  let key = null;
  let ref = null;
  let self = null;
  let source = null;

  // ref key 属性从config中取出赋值
  if (config != null) {
    if (hasValidRef(config)) {
      ref = config.ref;
    }
    if (hasValidKey(config)) {
      key = "" + config.key;
    }

    // self source 属性从config中取出赋值
    self = config.__self === undefined ? null : config.__self;
    source = config.__source === undefined ? null : config.__source;
    // 剩余的属性被添加到一个新的props对象中
    for (propName in config) {
      if (
        hasOwnProperty.call(config, propName) &&
        !RESERVED_PROPS.hasOwnProperty(propName)
      ) {
        props[propName] = config[propName];
      }
    }
  }

  // 子节点可以不只有一个参数, 这些子节点会被转移到新分配的props对象中
  const childrenLength = arguments.length - 2;
  if (childrenLength === 1) {
    props.children = children;
  } else if (childrenLength > 1) {
    const childArray = Array(childrenLength);
    for (let i = 0; i < childrenLength; i++) {
      childArray[i] = arguments[i + 2];
    }
    props.children = childArray;
  }

  // 处理默认props
  if (type && type.defaultProps) {
    const defaultProps = type.defaultProps;
    for (propName in defaultProps) {
      if (props[propName] === undefined) {
        props[propName] = defaultProps[propName];
      }
    }
  }

  return ReactElement(
    type,
    key,
    ref,
    self,
    source,
    ReactCurrentOwner.current,
    props
  );
}
```

ReactElement 属性

- `$$typeof` 是一个 Symbol 类型的变量，这个变量可以防止 XSS
- type 元素的类型
- key 组件的唯一标识
- ref 原生 dom 节点的引用
- props 传入组件的 props
- self 组件的实例
- source 代码来自的文件名和行数

```javascript
// ReactSymbols.js
const hasSymbol = typeof Symbol === "function" && Symbol.for;

export const REACT_ELEMENT_TYPE = hasSymbol
  ? Symbol.for("react.element")
  : 0xeac7;

// ReactElement.js
const ReactElement = function(type, key, ref, self, source, owner, props) {
  const element = {
    // This tag allows us to uniquely identify this as a React Element
    $$typeof: REACT_ELEMENT_TYPE,

    // Built-in properties that belong on the element
    type: type,
    key: key,
    ref: ref,
    props: props,

    // Record the component responsible for creating this element.
    _owner: owner
  };

  return element;
};

export function isValidElement(object) {
  return (
    typeof object === "object" &&
    object !== null &&
    object.$$typeof === REACT_ELEMENT_TYPE
  );
}
```

#### 虚拟 DOM 转换为真实 DOM

组件声明完成，我们需要调用`ReactDOM.render(element, container[, callback])` 渲染组件。在 React16+的版本中，引入了 Fiber 架构，在这一步中会将虚拟 Dom 转为 Fiber 结构类型。

```javascript
// ReactDOMLegacy.js
export function render(element, container, callback) {
  return legacyRenderSubtreeIntoContainer(
    null,
    element,
    container,
    false,
    callback
  );
}

function legacyCreateRootFromDOMContainer(
  container: DOMContainer,
  forceHydrate: boolean
): RootType {
  const shouldHydrate =
    forceHydrate || shouldHydrateDueToLegacyHeuristic(container);
  // 首先清除所有现有的内容
  if (!shouldHydrate) {
    let warned = false;
    let rootSibling;
    while ((rootSibling = container.lastChild)) {
      container.removeChild(rootSibling);
    }
  }

  return createLegacyRoot(
    container,
    shouldHydrate
      ? {
          hydrate: true
        }
      : undefined
  );
}

function legacyRenderSubtreeIntoContainer(
  parentComponent,
  children,
  container,
  forceHydrate,
  callback
) {
  let root: RootType = (container._reactRootContainer: any);
  let fiberRoot;
  if (!root) {
    // 初始化
    root = container._reactRootContainer = legacyCreateRootFromDOMContainer(
      container,
      forceHydrate
    );
    fiberRoot = root._internalRoot;
    if (typeof callback === "function") {
      const originalCallback = callback;
      callback = function() {
        const instance = getPublicRootInstance(fiberRoot);
        originalCallback.call(instance);
      };
    }
    // 初始化渲染不应该分批进行
    unbatchedUpdates(() => {
      updateContainer(children, fiberRoot, parentComponent, callback);
    });
  } else {
    fiberRoot = root._internalRoot;
    if (typeof callback === "function") {
      const originalCallback = callback;
      callback = function() {
        const instance = getPublicRootInstance(fiberRoot);
        originalCallback.call(instance);
      };
    }
    // 更新
    updateContainer(children, fiberRoot, parentComponent, callback);
  }
  return getPublicRootInstance(fiberRoot);
}
```

```javascript
// ReactFiberReconciler.js
export function updateContainer(element, container, parentComponent, callback) {
  // 当前fiber
  const current = container.current;
  // 获取当前更新开始时间
  const currentTime = requestCurrentTimeForUpdate();
  // 获取配置
  const suspenseConfig = requestCurrentSuspenseConfig();
  // 计算过期时间
  const expirationTime = computeExpirationForFiber(
    currentTime,
    current,
    suspenseConfig
  );

  // 获取上下文
  const context = getContextForSubtree(parentComponent);
  if (container.context === null) {
    container.context = context;
  } else {
    container.pendingContext = context;
  }

  // 创建更新任务
  const update = createUpdate(expirationTime, suspenseConfig);
  update.payload = { element };

  callback = callback === undefined ? null : callback;
  if (callback !== null) {
    update.callback = callback;
  }

  // 关联更新
  enqueueUpdate(current, update);
  // 调度
  scheduleWork(current, expirationTime);

  return expirationTime;
}
```
