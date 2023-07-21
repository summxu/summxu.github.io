---
title: 使用队列解决插队业务场景
date: 2019-03-1 09:02:36
tags:
    - JavaScript
---
为了达到操作变化的实时性，在多选择项的业务场景中，往往点击一次就会发送一次 HTTP 请求。
但同时又面临另外一个问题，在连续点击同一个选择项时，就会连续触发针对此项的删除或添加逻辑，进而连续发送 POST 或 DELETE 请求。
理想的状态是上一个请求结束后才开始下一个请求，但是网络请求是异步的、请求耗时是不可控的，也就有可能在此项的添加请求未完成前，删除请求先完成了。

我觉得用“插队”来描述这个场景真是再好不过了。

# 队列
既然有人要“插队”，我们就要定义一个规则：先进先出。
也就是数据结构中的“队列”了。

javascript 中队列的实现：
```javascript
// 实现1
var queue = []
// 进队
queue.push(1)
queue.push(2)
queue.push(3)
// 出队
queue.shift() // 1
queue.shift() // 2
queue.shift() // 3
// 实现2
var queue2 = []
// 进队
queue2.unshift(1)
queue2.unshift(2)
queue2.unshift(3)
// 出队
queue2.pop() // 1
queue2.pop() // 2
queue2.pop() // 3

```

# 实践
定义一个数组存放每一次点击的 HTTP 请求，此外不管你使用何种开发技术都应该有个发送请求的函数或库，如果是基于 promise 就更好了，这里简单用 XHR 代替。

```javascript
var requestQueue = []
var XHR = function(method, url, param) {}
```
点击操作的入口函数，先创建请求进队。
因为第二次请求必须要在第一次请求完成之后，所以只有队列中仅存在一个请求时才去触发更新。
```javascript
function myClick(method, url, param) {
    var len = requestQueue.push(XHR(method, url, param))
    if(len === 1) {
        update()
    }
}
```
执行的永远是队列中的第一个请求。
当一个请求完成后，就出队，队列中剩余的请求依次前进一个位置。
如果存在未完成的请求，继续调用更新操作。
```javascript
function update() {
    requestQueue[0].then(function(res){
        // request success
        requestQueue.shift()
        requestQueue.length && update()
    })
}
```