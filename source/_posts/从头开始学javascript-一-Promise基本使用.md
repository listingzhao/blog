---
title: 从头开始学javascript(一)-Promise基本使用
date: 2018-04-01 17:19:42
tags:
---
### 为什么使用Promise

#### 背景-回调地狱
在工作当中我们往往会遇到多个请求依赖的问题，可能就会有如下的写法

```javascript
$.post("/data1", { data: "data1"}, function(data1) {
  if(data.state === 'error') return
  $.post("/data2", { data: data1}, function(data2) {
    if(data2.state === 'error') return
    $.post("/data3", { data: data2}, function(data3) {
      console.log(data3)
    })
  })
})
```

这里只展示了三个请求间的互相依赖，如果业务复杂一些，需要有5，6个请求的话，代码看起来就有点恐怖了。。
Promise的出现正是为了解决这样的问题。

### 含义
Promise，就是一个对象，用来传递异步操作的消息，它代表了某个未来才会知道结果的事件（通常是一个异步操作）,并且这个事件提供统一的API，可供进一步处理。

### 特点

- 对象的状态不受外界影响。内部有3中状态：Pending（进行中）, Resolved(已完成), Rejected(已失败)，只有异步操作的结果可以决定当前的状态

- 一旦状态改变就不会再变，任何时候都可以得到这个结果

```javascript
let promise = new Promise(function(resolve, reject){
  // ...
  if(/* 异步操作成功*/){
    resolve(value)
  } else {
    reject(error)
  }
})

promise.then(function(value){
  // success
}, function(err){
  // reject
})

// 简单例子
function timeout(ms) {
  return new Promise((resolve, reject) => {
    setTimeout(resolve, ms, 'hello')
  })
}

timeout(100).then((value) => {
  console.log(value)
})
```

### 参考链接
[剖析Promise内部结构](https://github.com/xieranmaya/blog/issues/3)
[Promise原理浅析](http://imweb.io/topic/565af932bb6a753a136242b0)
