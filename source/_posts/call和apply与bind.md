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
