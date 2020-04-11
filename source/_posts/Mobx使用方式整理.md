---
title: Mobx使用方式整理
date: 2019-12-11 17:42:38
tags:
---

### Mobx 要点

#### 1. 定义状态并使其可观察

Mobx 可以为现有的数据结构(对象，数组和类)添加可观察的功能，使用@observable 装饰器给类属性添加注解。

```javascript
class Todo {
  id = Math.random();
  @observable title = "";
  @observable finished = false;
}
```

Computed values (计算值)
计算值是可以根据现有的状态和其它计算值衍生出的值。

```javascript
class TodoList {
  @observable todos = [];
  @computed get unfinishedTodoCount() {
    return this.todos.filter(todo => !todo.finished).length;
  }
}
```

[Demo 地址](https://codesandbox.io/s/mobx-computed-zf63i)

#### 2. 创建视图响应状态的变化 Reactions(反应)

Mobx 通过 observer 函数或装饰器来给组件添加响应式的功能，observer 来自 mobx-react 包。

```javascript
@observer
class TodoListView extends React.Components {
  render() {
    return (
      <div>
        {this.props.todoList.todos.map(todo => (
          <TodoView todo={todo} key={todo.id} />
        ))}
      </div>
    );
  }
}

const TodoView = observer(({ todo }) => (
  <li>
    <input
      type="checkbox"
      checked={todo.finished}
      onClick={() => (todo.finished = !todo.finished)}
    />
    {todo.title}
  </li>
));

const store = new TodoList();
ReactDOM.render(
  <TodoListView todoList={store} />,
  document.getElementById("mount")
);
```

##### 自定义 reactions

可以使用 autorun , reaction 和 when 函数即可简单创建自定义的 reactions。

[Demo 地址](https://codesandbox.io/s/mobx-autorun-k4ovw)

[Demo when 地址](https://codesandbox.io/s/mobx-when-xu5k4)

[Demo reaction 地址](https://codesandbox.io/s/mobx-reaction-pkrgm)

```javascript
// 对函数中使用的任何东西作出反应
autorun(() => {
  console.log("Tasks left: " + todos.unfinishedTodoCount);
});

// 对 length 和 title 的变化作出反应
reaction(() => {
  () => todos.map(todo => todo.title),
    titles => console.log("reaction 2:", titles.join(", "));
});

// when
when(predicate: () => boolean, effect?: () => void, options?)
```

#### 3. 更改状态 (action)

action 装饰器/函数遵循 javascript 中标准的绑定规则。action.bound 可以用来自动地将动作绑定到目标对象。(action.bound 不能与箭头函数一起使用)

只对修改状态的函数使用 action。

runInAction 是个简单的工具函数，它接收代码块并在(异步的)动作中执行。这对于即时创建和执行动作非常有用，例如在异步过程中。runInAction(f) 是 action(f)() 的语法糖。

##### 异步 action 方式

action 只能影响正在运行的函数，而无法影响当前函数调用的异步操作。action 包装/装饰器只会对当前运行的函数作出反应，而不会对当前运行函数所调用的函数（不包含在当前函数之内）作出反应，也就是说 promise 的 then 或 async 语句，并且在回调函数中某些状态改变了，这些回调函数也应该包装在 action 中。

[Demo 地址](https://codesandbox.io/s/async-action-6pln7)

在 promise 或 async 语句中可以使用 runInAction 包装回调函数，最简单的方式可以将回调函数包装成 action。

更好的方式是使用 flows 概念， flow 作为函数使用配合 function \*，yield 关键字，使用原理和 async 语句相同。

- Promises
- async / await
- flows

### 总结

mobx 的设计理念是基于数据驱动；源于应用状态的所有东西，都应该自动得到。比如 UI，数据序列化，服务通信
也就是说，只要知道哪些东西是状态相关的（源于应用状态），在状态发生变化时，就应该自动完成状态相关的所有事情，自动更新 UI，自动缓存数据，自动通知 server。

使用 mobx 的基本步骤

- 定义状态并使其可观察
- 创建视图响应状态的变化 Reactions(反应)
- 更改状态 (action)
