---
title: 手写coding题
date: 2024-01-06 09:49:38
tags:
  - JavaScript
---

## 手写Promise（没有考虑链式调用）
```JavaScript
  // 创建MyPromise类 
  class MyPromise {
    // 通过构造函数constructor，在执行这个类的时候需要传递一个执行器进去并立即调用
    constructor(executor) {
      this.status = 'pending'
      // 定义 fulfilled 情况下的返回值
      this.result = null
      // 定义 rejected 情况下的返回值
      this.reason = null
      // 定义回调函数数组
      this.onFulfilledCallbacks = []
      this.onRejectedCallbacks = []
      executor(this.resolve, this.reject)
    }
    // 定义resolve函数，改变状态和调用回调
    resolve = result => {
      if (this.status !== 'pending') return
      this.result = result
      this.status = 'fulfilled'
      // 执行数组里的 回调函数
      this.onFulfilledCallbacks.forEach(callback => callback(result))
    }
    // 定义reject函数，改变状态和调用回调
    reject = reason => {
      if (this.status !== 'pending') return
      this.reason = reason
      this.status = 'rejected'
      // 执行数组里的 回调函数
      this.onFulfilledCallbacks.forEach(callback => callback(reason))
    }
    // 定义then方法，接受两个回调函数
    then = (onFulfilled, onRejected) => {
      // 判断接受的回调函数，并可以为空给默认值
      onRejected = typeof onRejected === 'function' ? onRejected : reason => { throw reason }
      onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : result => result
      if (this.status === 'fulfilled') {
        // fulfilled状态，调用第一个 fulfilled回调函数
        // 添加到宏任务，尽量晚于执行器后
        setTimeout(() => { onFulfilled(this.result) });
      } else if (this.status === 'rejected') {
        // rejected状态，调用第二个 rejected回调函数
        // 添加到宏任务，尽量晚于执行器后
        setTimeout(() => { onRejected(this.reason) });
      } else if (this.status === 'pending') {
        // 如果执行器中为异步（执行时机晚）不会立即改变状态，此时调用then一定是pending
        // 这个时候可以把回调函数保存到数组里，等待resolve/reject调用执行then
        this.onFulfilledCallbacks.push(onFulfilled)
        this.onRejectedCallbacks.push(onRejected)
      }
    }

    catch = (onRejected) => {
      setTimeout(() => { onRejected(this.reason) });
    }
  }

  new MyPromise((resolve, reject) => {
    setTimeout(() => { resolve('2秒之后执行then') }, 200);
  }).then(res => { console.log(res) })
```

## 手写节流，防抖
- 节流
```JavaScript
function throttle(fn) {
  let canRun = true
  return function () {
    if(canRun === false) return
    canRun = false
    setTimeout(() => {
      fn.apply(this, arguments)
      canRun = true
    }, 500)
  }
}
```
- 防抖
```JavaScript
function debounce(fn) {
  let timer = null
  return function() {
    clearTimeout(timer)
    timer = setTimeout(() => {
      fn.apply(this, arguments)
    }, 500)
  }
}
```

## 观察者模式 && 发布订阅模式
- 发布订阅模式（发布订阅模式可以通知更新部分订阅者，中间有一层changemanager
```JavaScript
class EventBus {
  constructor() {
    this.eventList = {}
  }
  on(eventName, callBack) {
    if(!this.eventList[eventName]) this.eventList[eventName] = []
    this.eventList[eventName].push(callBack)
  }
  emit(eventName, ...args) {
    if(!this.eventList[eventName]) return
    this.eventList[eventName].forEach(callback => {
      callback.call(this, ...args)
    })
  }
}
const eventBus = new EventBus()
eventBus.on('update', msg => { console.log(msg) })
eventBus.emit('update', 'hello')
```
- 观察者模式（观察者中，主题和观察对象是互相感知的）
```JavaScript
class Observer {
  constructor() {
    this.watcherList = []
  }
  addWatcher(watcher) {
    this.watcherList.push(watcher)
  }
  notify(...args) {
    this.watcherList.forEach(watcher => {
      watcher.update(...args)
    })
  }
}
class Watcher {
  constructor(name) {
    this.name = name
  }
  update(msg) {
    console.log(`my name is ${this.name}, i update ${msg}`)
  }
}
const ob = new Observer()
const aa = new Watcher('aa')
const bb = new Watcher('bb')
ob.addWatcher(aa)
ob.addWatcher(bb)
ob.notify('hello')
```

## promiseAll
```JavaScript
function promiseAll(promiseList) {
  const results = []
  return new Promise(async (resolve, reject) => {
    for (let i = 0; i < promiseList.length; i++) {
      const item = promiseList[i]
      try {
        results.push(await item)
      } catch (error) {
        reject(error)
      }
    }
    resolve(results)
  });
}
```