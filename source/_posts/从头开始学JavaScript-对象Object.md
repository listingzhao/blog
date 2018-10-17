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
- 借用构造函数