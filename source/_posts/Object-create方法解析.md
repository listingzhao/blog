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

使用 Object.create 的 propertyObject 参数

``` javascript
var o;

// 创建一个原型为null的空对象
o = Object.create(null);


o = {};
// 以字面量方式创建的空对象就相当于:
o = Object.create(Object.prototype);


o = Object.create(Object.prototype, {
  // foo会成为所创建对象的数据属性
  foo: {
    writable:true,
    configurable:true,
    value: "hello"
  },
  // bar会成为所创建对象的访问器属性
  bar: {
    configurable: false,
    get: function() { return 10 },
    set: function(value) {
      console.log("Setting `o.bar` to", value);
    }
  }
});


function Constructor(){}
o = new Constructor();
// 上面的一句就相当于:
o = Object.create(Constructor.prototype);
// 当然,如果在Constructor函数中有一些初始化代码,Object.create不能执行那些代码


// 创建一个以另一个空对象为原型,且拥有一个属性p的对象
o = Object.create({}, { p: { value: 42 } })

// 省略了的属性特性默认为false,所以属性p是不可写,不可枚举,不可配置的:
o.p = 24
o.p
//42

o.q = 12
for (var prop in o) {
   console.log(prop)
}
//"q"

delete o.p
//false

//创建一个可写的,可枚举的,可配置的属性p
o2 = Object.create({}, {
  p: {
    value: 42,
    writable: true,
    enumerable: true,
    configurable: true
  }
});
```
