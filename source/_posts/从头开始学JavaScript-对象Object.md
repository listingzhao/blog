---
title: 从头开始学JavaScript-对象Object
date: 2018-10-17 10:52:24
tags:
---

## 对象属性
###1. 属性类型
- 数据属性 Configurable Enumberable Writable Value
- 访问器属性 Configurable Enumberable  Get Set

###2.特殊方法
- Object.defineProperty()
- Object.defineProperties()
- Object.getOwnPropertyDescriptor() 读取对象属性的特性 （根据不同属性返回不同对象）

## 创建对象
- 工厂模式
- 构造函数模式
1. 构造函数主要问题在于对象的每个方法都要在每个实例上重新创建一遍，因此不同实例上的同名函数是不相等的。
- 原型模式
1. 原型对象
- Fun.prototype.isPrototypeOf(obj)   // true or false
- Object.getPrototypeOf(obj) // 返回原型对象
- Object.hasOwnProperty()  // 判断属性是否存在实例中
- Object.keys()  // 获取对象可枚举属性字符串数组
存在的问题
1. 省略了构造传参，初始化数据不方便
2. 由于本身共享特性导致的引用值修改问题
- 组合使用构造函数模式和原型模式
- 寄生构造函数模式 (构造函数返回对象实例)
- 稳妥构造函数模式 (和寄生类似，区别在于新创建的实例不引用this)

## 继承

- 原型链 （实践中很少单独使用）
问题：
1. 创建子类型的实例时不能向超类型的构造函数中传递参数
2. 原型中包含引用类型值的问题
- 借用构造函数 （实践中很少单独使用）
1. 传递参数
```javascript
function SuperType(name) {
  this.name = name
}
function SubType() {
  SuperType.call(this， "Nicholas")
  this.age = 14
}
var instance = new SubType();
console.log(instance.name)
console.log(instance.age)
```
2. 解决原型中包含引用类型值带来的问题
问题：
1. 无法避免构造函数模式存在的问题-函数复用
2. 原型定义的方法，对子类型不可见，结果所有类型都只能使用构造函数模式

- 组合继承
将原型继承和借用构造函数组合在一起使用
```javascript
function SuperType(name) {
  this.name = name
  this.colors = ['red', 'blue', 'green']
}
SuperType.prototype.sayName = function() {
  alert(this.name)
}
function SubType(age, name) {
  SuperType.call(this, name)
  this.age = age
}

SubType.prototype = new SuperType();

SubType.prototype.sayAge = function() {
  alert(this.age)
}

var instance1 = new SubType(17, 'Nicholas');
instance1.push('black')
console.log(instance1.colors)  // 'red, blue, green, black'
instance1.sayName()
instance1.sayAge()

```

- 原型式继承
```javascript
function object(o) {
  function F() {}
  F.prototype = o
  return new F()
}

var person = {
  name: 'Nicholads',
  friends: ['Shelby', 'Court', 'Van']
}

var aPerson = object(person)
aPerson.name = 'Greg'
aPerson.friends.push('Rob')

```

- 寄生式继承
问题：
1. 和构造函数模式类似，不能函数复用
```javascript
function createAnother(o) {
  var clone = object(o)    // 调用函数创建新对象
  clone.sayHi = function() {
    alert('hi')
  }
  return clone    
}

var person = {
  name: 'Nicholads',
  friends: ['Shelby', 'Court', 'Van']
}

var aPerson = createAnother(person)
aPerson.sayHi()

```

- 寄生组合式继承
```javascript

function SuperType(name) {
  this.name = name
  this.colors = ['red', 'blue', 'green']
}

function SubType(age, name) {
  SuperType.call(this, name)   // 第二次调用
  this.age = age
}

SubType.prototype = new SuperType();   // 第一次调用
SubType.prototype.constructor = SubType;
SubType.prototype.sayAge = function () {
  alert(this.age)
}

function inheritPrototype(subType, superType) {
  var prototype = object(superType)
  prototype.constructor = subType
  subType.prototype = prototype
}

function SuperType(name) {
  this.name = name
  this.colors = ['red', 'blue', 'green']
}

function SubType(age, name) {
  SuperType.call(this, name)
  this.age = age
}

inheritPrototype(SubType, SuperType);

SubType.prototype.sayAge = function () {
  alert(this.age)
}
```