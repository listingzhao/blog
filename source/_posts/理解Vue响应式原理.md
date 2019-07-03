---
title: 理解Vue响应式原理
date: 2018-04-02 23:37:28
tags:
---

```javascript
const v = new Vue({
  data: {
    a:1,
    b:2
  }
})
v.$watch('a',() => console.log('watch 成功了!'))

setTimeout(() => {
  v.a = 6
}, 2000)

// 发布订阅 依赖收集追踪
export class Dep {
  constructor() {
    this.subs = []
  }

  addSub (sub) {
    // 订阅
    this.subs.push(sub)
  }

  depend () {
    if (Dep.target) {
      Dep.target.addDep(this)
    }
  }

  notify() {
    // 通知
    const subs = this.subs.slice()
    for(let i=0; i<subs.length; i++){
      subs[i].update()
    }
  }
}

Dep.target = null

const targetStack = []

export class Watcher {
  constructor (vm, expOrFn, cb) {
    this.vm = vm
    this.cb = cb
    this.expOrFn = expOrFn
    this.value = this.get()
  }

  update() {
    this.run()
  }

  run() {
    const value = this.get()
    if(value !== this.value){
      // set new value
      const oldValue = this.value
      this.value = value
      this.cb.call(this.vm, value, oldValue)
    }
  }

  get() {
    pushTarget(this)
    let value = this.vm._data[this.expOrFn]
    popTarget()
    return value
  }

  addDep (dep) {
    dep.addSub(this)
  }
}

export function pushTarget (target) {
  targetStack.push(target)
  Dep.target = target
}

export function popTarget () {
  targetStack.pop()
  Dep.target = targetStack[targetStack.length - 1]
}

export class Observer {
  constructor(value) {
    this.value = value
    this.dep = new Dep()
    this.walk(value)
  }

  /**
   *遍历每个属性并将其转换为getter / setter。此方法只应值类型是对象时调用。
   */
  walk(obj){
    const keys = Object.keys(obj)
    for(let i = 0; i < keys.length; i++){
      defineReactive(obj, keys[i])
    }
  }
}

export function defineReactive(obj, key, val) {
  const dep = new Dep()
  if(arguments.length === 2) {
    val = obj[key]
  }
  let childOb = observe(val)
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter() {
      if(Dep.target) { // 判断是否是watcher
        dep.depend()
        if(childOb){
          childOb.dep.depend()
          // 暂不考虑数组
        }
      }
      return val
    },
    set: function reactiveSetter(newVal) {
      val = newVal
      childOb = observe(val)
      dep.notify() //通知
    }
  })
}

/**
*试图为一个值,创建一个观察者实例返回新观察员如果成功地观察到,或者如果价值已经有了一个现有的观察者。
*/
export function observe(value) {
  let ob = new Observer(value)
  return ob
}

export function proxy(target, sourceKey, key) {
  Object.defineProperty(target, key, {
      configurable: true,
      enumerable: true,
      get: function proxyGetter () {
        return this[sourceKey][key]
      },
      set: function proxySetter (val) {
        this[sourceKey][key] = val
      }
    })
}

export class Vueb {
  constructor(options ={}){
    this.$options = options
    let data = this._data = this.$options.data
    // proxy data on instance
    const keys = Object.keys(data)
    let i = keys.length
    while(i--) {
      let key = keys[i]
      proxy(this, `_data`, key)
    }
    // observe data
    observe(data)
  }

  $watch(expOrFn, cb, options) {
    new Watcher(this, expOrFn, cb)
  }

}

const v2 = new Vueb({
  data: {
    a:1,
    b:2
  }
})

v2.$watch('a', (value, oldValue) => {
  console.log('haha Vueb watch 成功了!')
  console.log('value:', value)
  console.log('oldValue:', oldValue)
})

setTimeout(() => {
  v2.a = 6
}, 2000)
```
