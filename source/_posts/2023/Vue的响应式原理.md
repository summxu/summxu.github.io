---
title: Vue的响应式原理
date: 2023-05-07 09:07:32
tags:
---

## vue2使用Object.defineProperty实现响应式

- 遍历属性，对每一个属性的值用Object.defineProperty进行getter和setter的改造；

```TypeScript
//get
function track(data, key){
  console.log('get data:', key)
}
// set
function trigger(data, key, value){
  console.log('set data:', key,'-', value)
}

function observe(data){
  if(!data || typeof data !== 'object') return
  for(let key in data){
    let value = data[key]
    Object.defineProperty(data, key, {
      enumerable: true,
      configurable: true,
      get(){
        track(data, key)
        return value
      },
      set(newVal){
        trigger(data, key, newVal)
        value = newVal
      }
    })
    if(typeof value === 'object'){
      observe(value)
    }
  }
}

var obj = {
  name: 'win',
  family: [2,5,8]
}

observe(obj)

obj.name = 'ten'
obj.family[3] = 6 //不是响应式，没有被Object.defineProperty包装一下
obj.age = 18 //不是响应式，没有被Object.defineProperty包装一下
console.log(obj, 'obj')
```

Vue3相对于Vue2响应式原理也发生了变化，由原先的 `Object.defineproperty` 改成了使用 `Proxy` 替代。`Proxy` 相对于 `Object.defineproperty` 有以下几个优化点：

-   对象新增属性不再需要手动 `$set` 添加响应式，`Proxy` 默认会监听动态添加属性和属性的删除等操作。
-   消除无法监听数组索引，length 属性等等，不再需要在数组原型对象上重写数组的方法。
-   `Object.defineproperty` 是劫持所有对象属性的 `get`/`set` 方法,需要遍历递归去实现，`Proxy` 是代理整个对象。
-   Vue2 只能拦截对象属性的 `get` 和 `set` 操作,而 `Proxy` 拥有 `13` 种拦截方法。

## Vue3中的响应式原理
实现原理：

通过Proxy（代理）： 拦截对象中任意属性的变化，包括：属性值的读写，属性的增加，属性的删除等。

通过Reffect（反射）： 对源对象的属性进行操作

### Proxy

-   Proxy对象对于创建一个对象的代理，也可以理解成在对象前面设了一层拦截，**可以实现基本操作的拦截和一些自定义操作（比如一些赋值、属性查找、函数调用等）；**
-   用法：`var proxy = new Proxy(target, handler);`  
    **target：**目标函数（即进行改造的函数）；  
    **handler：**一些自定义操作（比如vue中getter和setter的操作）；

### Reflect

-   Reflect是es6为操作对象而提供的新API，设计它的目的有：  
    ① 把Object对象上一些明显属于语言内部的方法放到Reflect对象身上，比如Object.defineProperty；  
    ② 修改了某些object方法返回的结果；  
    ③ 让Object操作都变成函数行为；  
    ④ Reflect对象上的方法和Proxy对象上的方法一一对应，这样就可以让Proxy对象方便地调用对应的Reflect方法；
-   `Reflect.get(target, propertyKey, receiver)`：**等价于`target[propertyKey]`，**Reflect.get方法查找并返回target对象的propertyKey属性，如果没有该属性，则返回undefined。
-   `Reflect.set(target, propertyKey, value, receiver)`：**等价于`target[propertyKey] = value`，**Reflect.set方法设置target对象的propertyKey属性等于value。

### Proxy和Reflect的使用
```TypeScript
const obj = {
  name: 'win'
}

const handler = {
  get: function(target, key){
    console.log('get--', key)
    return Reflect.get(...arguments)  
  },
  set: function(target, key, value){
    console.log('set--', key, '=', value)
    return Reflect.set(...arguments)
  }
}

const data = new Proxy(obj, handler)
data.name = 'ten'
console.log(data.name,'data.name22')
```

## 使用Proxy和Reflect完成响应式
```TypeScript
function track(target, key){
  console.log('get--', key)
}

function trigger(target, key, value){
  console.log('set--', key, '=', value)
}

function reactive(obj){
  const handler = {
    get(target, key, receiver){
      track(target, key)
      const value = Reflect.get(...arguments)
      if(typeof value === 'object'){
        return reactive(value)
      }else{
        return value
      }
    },
    set(target, key, val, receiver){
      trigger(target, key, val)
      return Reflect.set(...arguments)
    }
  }
  
  return new Proxy(obj, handler)
}

const obj = {
  name: 'win'
}

const data = reactive(obj)

data.list = [5] //响应式  这两句话是执行Reflect.set()
data.age = 18 //响应式
console.log(data, 'data111')
```