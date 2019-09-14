---
title: 正确使用Vue指令的钩子函数
date: 2019-03-24 15:28:24
tags:
    - Vue
---
在 Vue 中可以把一系列**复杂的操作**包装为一个指令。

> **什么是复杂的操作？**
> 我的理解是：复杂逻辑功能的包装、违背数据驱动的 DOM 操作以及对一些 Hack 手段的掩盖等。我们总是期望以操作数据的形式来实现功能逻辑。

# 钩子函数
对于自定义指令的定义，Vue2 有 5 个可选的钩子函数。

* bind: 只调用一次，指令第一次绑定到元素时调用，用这个钩子函数可以定义一个在绑定时执行一次的初始化动作。
* inserted: 被绑定元素插入父节点时调用（父节点存在即可调用，不必存在于 document 中）。
* update: 被绑定元素所在的模板更新时调用，而不论绑定值是否变化。
* componentUpdated: 被绑定元素所在模板完成一次更新周期时调用。
* unbind: 只调用一次，指令与元素解绑时调用。
接下来，定义一个简单的指令以验证这些钩子函数的触发时机。
```html
<div id="app">
  <my-comp v-if="msg" :msg="msg"></my-comp>
  <button @click="update">更新</button>
  <button @click="uninstall">卸载</button>
  <button @click="install">安装</button>
</div>
```
```javascript
Vue.directive('hello', {
    bind: function (el) {
        console.log('bind')
    },
    inserted: function (el) {
        console.log('inserted')
    },
    update: function (el) {
        console.log('update')
    },
    componentUpdated: function (el) {
        console.log('componentUpdated')
    },
    unbind: function (el) {
        console.log('unbind')
    }
})
var myComp = {
    template: '<h1 v-hello>{{msg}}</h1>',
    props: {
        msg: String
    }
}
new Vue({
    el: '#app',
    data: {
        msg: 'Hello'
    },
    components: {
        myComp: myComp
    },
    methods: {
        update: function () {
            this.msg = 'Hi'
        },
        uninstall: function () {
            this.msg = ''
        },
        install: function () {
            this.msg = 'Hello'
        }
    }
})
```
## 页面加载时
```
bind
inserted
```
## 组件更新时
点击“更新”按钮，更改数据触发组件更新。

```
update
componentUpdated
```
## 卸载组件时
点击“卸载”按钮，数据置空否定判断以触发组件卸载。

`unbind`
## 重新安装组件时
点击“安装”按钮，数据赋值肯定判断以触发组件重新安装。
```
bind
inserted
```
## 区别
从案例的运行中，对 5 个钩子函数的触发时机有了初步的认识。存疑的也就是`bind`和`inserted`、`update`和`componentUpdated`的区别了。

### bind 和 inserted
据文档所说，插入父节点时调用 inserted，来个测试。
```javascript
bind: function (el) {
    console.log(el.parentNode)  // null
    console.log('bind')
},
inserted: function (el) {
    console.log(el.parentNode)  // <div id="app">...</div>
    console.log('inserted')
}
```
分别在两个钩子函数中输出父节点：**bind 时父节点为 null，inserted 时父节点存在。**

### **update** 和 **componentUpdated**
关于这两个的介绍，从字眼上看感觉是组件更新周期有关，继续验证。

```javascript
update: function (el) {
    console.log(el.innerHTML)   // Hello
    console.log('update')
},
componentUpdated: function (el) {
    console.log(el.innerHTML)   // Hi
    console.log('componentUpdated')
}
```
没毛病，**update 和 componentUpdated 就是组件更新前和更新后的区别。**

## 结论
文档说的没错…😒
[Demo](https://jsfiddle.net/imys/twbv0sov/1/)

## 最佳实践
根据需求的不同，我们要选择恰当的时机去初始化指令、更新指令调用参数以及释放指令存在时的内存占用等。

比较常见的场景是：用指令包装一些无依赖的第三方库以扩展组件功能。而一个健壮的库通常会包含：初始化实例、参数更新和释放实例资源占用等操作。
```javascript
Vue.directive('hello', {
    bind: function (el, binding) {
        // 在 bind 钩子中初始化库实例
        // 如果需要使用父节点，也可以在 inserted 钩子中执行
        el.__library__ = new Library(el, binding.value)
    },
    update: function (el, binding) {
        // 模版更新意味着指令的参数可能被改变，这里可以对库实例的参数作更新
        // 酌情使用 update 或 componentUpdated 钩子
        el.__library__.setOptions(Object.assign(binding.oldValue, binding.value))
    },
    unbind: function (el) {
        // 释放实例
        el.__library__.destory()
    }
})
```