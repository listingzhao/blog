---
title: Object.create方法解析
date: 2017-09-22 11:26:19
tags:
---

### create 方法

Object.create() 方法会使用指定的原型对象及其属性去创建一个新的对象。

> Object.create(proto, [ propertiesObject ])

#### 参数

`proto`
一个对象，应该是新创建的对象的原型。

`propertiesObject`
可选。该参数对象是一组属性与值，该对象的属性名称将是新创建的对象的属性名称，值是属性描述符（这些属性描述符的结构与Object.defineProperties()的第二个参数一样）。注意：该参数对象不能是 undefined，另外只有该对象中自身拥有的可枚举的属性才有效，也就是说该对象的原型链上属性是无效的。

### 返回值

返回一个新对象。在指定原型对象上添加新属性后的新对象

### 抛出异常

如果 propertiesObject 参数不是 null 也不是对象，则抛出一个 TypeError 异常。

### 例子

使用 Object.create 实现类式继承

``` javascript
// Shape - superclass
function Shape () {
  this.x = 0
  this.y = 0
}

Shape.prototype.move = function (x, y) {
  this.x += x
  this.y += y
  console.info('Shape moved.')
}

// Rectangle - subclass
function Rectangle () {
  Shape.call(this) // call super constructor.
}

// subclass extends superclass
Rectangle.prototype = Object.create(Shape.prototype)
Rectangle.prototype.constructor = Rectangle

var rect = new Rectangle()

console.log('Is rect an instance of Rectangle?',
  rect instanceof Rectangle) // true
console.log('Is rect an instance of Shape?',
  rect instanceof Shape) // true

rect.move(1, 1) // Outputs, "Shape moved."
```

继承到多个对象,则可以使用混入的方式。

``` javascript
function MyClass() {
     SuperClass.call(this);
     OtherSuperClass.call(this);
}

// inherit one class
MyClass.prototype = Object.create(SuperClass.prototype);
// mixin another
Object.assign(MyClass.prototype, OtherSuperClass.prototype);
// re-assign constructor
MyClass.prototype.constructor = MyClass;

MyClass.prototype.myMethod = function() {
     // do a thing
};
```
