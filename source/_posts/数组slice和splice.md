---
title: 数组slice和splice
date: 2017-09-18 17:46:49
tags:
---

### slice 方法
slice方法将数组的一部分的浅拷贝返回到从头到尾选择的新的数组对象（不包括结束）。原始数组不会被修改。

``` javascript
var a = ['zero', 'one', 'two', 'three']
var sliced = a.slice(1, 3)

console.log(a)      // ['zero', 'one', 'two', 'three']
console.log(sliced) // ['one', 'two']
```

#### 语法
> arr.slice()
  arr.slice(begin)
  arr.slice(begin, end)

#### 使用slice

``` javascript
// Using slice, create newCar from myCar.
var myHonda = { color: 'red', wheels: 4, engine: { cylinders: 4, size: 2.2 } }
var myCar = [myHonda, 2, 'cherry condition', 'purchased 1997']
var newCar = myCar.slice(0, 2)

// Display the values of myCar, newCar, and the color of myHonda
//  referenced from both arrays.
console.log('myCar = ' + JSON.stringify(myCar))
console.log('newCar = ' + JSON.stringify(newCar))
console.log('myCar[0].color = ' + myCar[0].color)
console.log('newCar[0].color = ' + newCar[0].color)

// Change the color of myHonda.
myHonda.color = 'purple';
console.log('The new color of my Honda is ' + myHonda.color)

// Display the color of myHonda referenced from both arrays.
console.log('myCar[0].color = ' + myCar[0].color)
console.log('newCar[0].color = ' + newCar[0].color)
```
