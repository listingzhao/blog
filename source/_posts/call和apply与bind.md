---
title: call和apply与bind
date: 2017-09-20 16:19:17
tags:
---

### call方法

call() 方法调用一个函数, 其具有一个指定的this值和分别地提供的参数(参数的列表)。

> fun.call(thisArg[, arg1[, arg2[, ...]]])

#### 参数
`thisArg`
在 `fun` 函数运行时指定的this值。需要注意的是，指定的this值并不一定是该函数执行时真正的this值，如果这个函数处于非严格模式下，则指定为null和undefined的this值会自动指向全局对象(浏览器中就是window对象)，同时值为原始值(数字，字符串，布尔值)的this会指向该原始值的自动包装对象。

`arg1, arg2, ...`
指定的参数列表。

### 描述
可以让call()中的对象调用当前对象所拥有的function。你可以使用call()来实现继承：写一个方法，然后让另外一个新的对象来继承它（而不是在新对象中再写一次这个方法）。

### 示例
使用call方法调用父构造函数

``` javascript
function Product (name, price) {
  this.name = name
  this.price = price
}

function Food (name, price) {
  Product.call(this, name, price)
  this.category = 'food'
}

function Toy (name, price) {
  Product.call(this, name, price)
  this.category = 'toy'
}

var cheese = new Food('feta', 5)
var fun = new Toy('robot', 40)
```

使用call方法调用匿名函数

``` javascript
var animals = [
  { species: 'Lion', name: 'King' },
  { species: 'Whale', name: 'Fail' }
]

for (let i = 0; i < animals.length; i++) {
  (function (i) {
    this.print = function () {
      console.log('#' + i + ' ' + this.species + ':' + this.name)
    }
    this.print()
  }).call(animals[i], i)
}
```

使用call方法调用函数并且指定上下文的'this'
``` javascript

function greet () {
  var reply = [this.person, 'Is An Awesome', this.role].join(' ')
  console.log(reply)
}

var i = {
  person: 'Douglas Crockford', role: 'Javascript Developer'
}

greet.call(i)
```

### apply方法

apply() 方法调用一个函数, 其具有一个指定的this值，以及作为一个数组（或类似数组的对象）提供的参数。

> fun.apply(thisArg, [argsArray])

#### 参数
`thisArg`
在 `fun` 函数运行时指定的this值。需要注意的是，指定的this值并不一定是该函数执行时真正的this值，如果这个函数处于非严格模式下，则指定为null和undefined的this值会自动指向全局对象(浏览器中就是window对象)，同时值为原始值(数字，字符串，布尔值)的this会指向该原始值的自动包装对象。

`argsArray`
一个数组或者类数组对象，其中的数组元素将作为单独的参数传给 `fun` 函数。如果该参数的值为null 或 undefined，则表示不需要传入任何参数。

### 描述
在调用一个存在的函数时，你可以为其指定一个 this 对象。 this 指当前对象，也就是正在调用这个函数的对象。 使用 apply， 你可以只写一次这个方法然后在另一个对象中继承它，而不用在新对象中重复写该方法。

apply 与 call() 非常相似，不同之处在于提供参数的方式。apply 使用参数数组而不是一组参数列表。apply 可以使用数组字面量，如 `fun.apply(this, ['eat', 'bananas'])`，或数组对象， 如  `fun.apply(this, new Array('eat', 'bananas'))`。

### 示例
使用apply来链接构造器

``` javascript
Function.prototype.construct = function (aArgs) {
  var oNew = Object.create(this.prototype)
  this.apply(oNew, aArgs)
  return oNew
}

function MyConstructor () {
  for (var nProp = 0; nProp < arguments.length; nProp++) {
    this['property' + nProp] = arguments[nProp]
  }
}

var myArray = [4, 'Hello world!', false]
var myInstance = MyConstructor.construct(myArray)

console.log(myInstance.property1)                // logs "Hello world!"
console.log(myInstance instanceof MyConstructor) // logs "true"
console.log(myInstance.constructor)
```

使用apply和内置函数

``` javascript

function minOfArray (arr) {
  var min = Infinity
  var QUANTUM = 32768

  for (var i = 0, len = arr.length; i < len; i += QUANTUM) {
    var submin = Math.min.apply(null, arr.slice(i, Math.min(i + QUANTUM, len)))
    min = Math.min(submin, min)
  }

  return min
}

var min = minOfArray([5, 6, 2, 3, 7])
```

### bind方法

bind()方法创建一个新的函数, 当被调用时，将其this关键字设置为提供的值，在调用新函数时，在任何提供之前提供一个给定的参数序列。

> fun.bind(thisArg[, arg1[, arg2[, ...]]])

#### 参数
`thisArg`
当绑定函数被调用时，该参数会作为原函数运行时的 this 指向。当使用new 操作符调用绑定函数时，该参数无效。

`arg1, arg2, ...`
当绑定函数被调用时，这些参数将置于实参之前传递给被绑定的方法。

#### 返回值
返回由指定的this值和初始化参数改造的原函数拷贝。

### 描述
