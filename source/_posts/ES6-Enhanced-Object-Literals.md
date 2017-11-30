---
title: ES6 Enhanced Object Literals
date: 2017-11-29 14:25:53
tags:
---

### ES6 字面量语法

``` javascript
var obj = {
    // Sets the prototype. "__proto__" or '__proto__' would also work.
    __proto__: theProtoObj,
    // Computed property name does not set prototype or trigger early error for
    // duplicate __proto__ properties.
    ['__proto__']: somethingElse,
    // Shorthand for ‘handler: handler’
    handler,
    // Methods
    toString() {
     // Super calls
     return "d " + super.toString();
    },
    // Computed (dynamic) property names
    [ "prop_" + (() => 42)() ]: 42
};
```

### 转换之后的代码

``` javascript
'use strict'

var _obj, _obj2

var _get = function get (object, property, receiver) {
  if (object === null) object = Function.prototype
  var desc = Object.getOwnPropertyDescriptor(object, property)
  if (desc === undefined) {
    var parent = Object.getPrototypeOf(object)
    if (parent === null) {
      return undefined
    } else {
      return get(parent, property, receiver)
    }
  } else if ('value' in desc) {
    return desc.value
  } else {
    var getter = desc.get
    if (getter === undefined) {
      return undefined
    }
    return getter.call(receiver)
  }
}

function _defineProperty (obj, key, value) {
  if (key in obj) {
    Object.defineProperty(obj, key, {
      value: value,
      enumerable: true,
      configurable: true,
      writable: true
    })
  } else {
    obj[key] = value
  }
  return obj
}

var obj = (_obj = ((_obj2 = {
  // Sets the prototype. "__proto__" or '__proto__' would also work.
  __proto__: theProtoObj
}),
(_obj2['__proto__'] = somethingElse),
_defineProperty(_obj2, 'handler', handler),
_defineProperty(_obj2, 'toString', function toString () {
  // Super calls
  return (
    'd ' +
    _get(_obj.__proto__ || Object.getPrototypeOf(_obj), 'toString', this).call(
      this
    )
  )
}),
_defineProperty(
  _obj2,
  'prop_' +
    (function () {
      return 42
    })(),
  42
),
_obj2))
```
