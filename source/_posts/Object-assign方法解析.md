---
title: Object-assign方法解析
date: 2017-09-29 10:25:42
tags:
---

### assign 方法

Object.assign() 方法用于将所有可枚举的属性的值从一个或多个源对象复制到目标对象。它将返回目标对象。

> Object.assign(target, ...sources)

#### 参数

`target`
目标对象

`sources`
(多个)源对象

### 返回值
目标对象

### 描述

如果目标对象中的属性具有相同的键，则属性将被源中的属性覆盖。后来的源的属性将类似地覆盖早先的属性。
Object.assign 方法只会拷贝源对象自身的并且可枚举的属性到目标对象身上。
注意，在属性拷贝过程中可能会产生异常，比如目标对象的某个只读属性和源对象的某个属性同名，这时该方法会抛出一个 TypeError 异常，拷贝过程中断，已经拷贝成功的属性不会受到影响，还未拷贝的属性将不会再被拷贝。
注意， Object.assign 会跳过那些值为 null 或 undefined 的源对象。

### 示例

复制一个object

``` javascript
var obj = { a: 1 }
var copy = Object.assign({}, obj)
console.log(copy) // { a: 1 }
```

深度拷贝问题

针对深度拷贝，需要使用其他方法，因为 Object.assign() 拷贝的是属性值。假如源对象的属性值是一个指向对象的引用，它也只拷贝那个引用值。
