---
title: 如何为Canvas中特定图形绑定事件？
date: 2019-02-14 18:47:15
tags:
    - Canvas
    - JavaScript
---

Canvas 本身也属于 HTMLElement，自然也是支持各种事件绑定的。
但绘制在其中的图形并不作为其子元素存在，这就不能方便的为 Canvas 中的某个特定图形去绑定事件。

我们都知道 js 中的事件委托，将事件绑定到父节点上，待到父节点响应事件时，动态判断当前响应元素为目标子节点时再执行对应的操作。

这个思想同样也可以用在 Canvas 上，只需要为 `canvas` 元素绑定事件，事件响应时判断当前鼠标位置处于哪个图形之上，执行对应的操作。

# isPointInPath

`context.isPointInPath(x, y);`

理论上讲，想要知道一个点是否处于一个图形之中，现成的算法应该是有很多了。不过难得 canvas 本身就提供了这样的函数，用来判断一个点是否处于当前路径中。

```javascript
var c = document.getElementById('canvas');
var ctx = c.getContext('2d');
ctx.rect(0, 0, 200, 200);
console.log(ctx.isPointInPath(50, 100))  // true
```
就像这样，当你创建一个矩形时，就会产生一个路径，此时就可以调用该方法去判断一个点是否存在于该路径。

产生路径的函数还有其他，比如：`lineTo()`、`clip()`、`arc()`、`arcTo()` 等。

# 实现图形的事件绑定
先来个简单的饼图吧。
```javascript
var canvas = document.getElementById('c');
var ctx = canvas.getContext('2d');
var r = canvas.width / 2;
ctx.beginPath();
ctx.arc(r, r, r, 0, Math.PI * 1);
ctx.fillStyle = '#2196f3'; //蓝色
ctx.fill();
ctx.beginPath();
ctx.arc(r, r, r, Math.PI * 1, Math.PI * 2);
ctx.fillStyle = '#f44336'; //红色
ctx.fill();
function isInPath (x, y){
    ctx.arc(r, r, r, 0, Math.PI * 1);
    return ctx.isPointInPath(x, y);
}
canvas.addEventListener('click', function(e){
    if(isInPath(e.offsetX, e.offsetY)) {
        console.log('hello')
    }
})
```
现在创建一个红蓝拼接的饼图，`isInPath` 方法判断一个点是否处于蓝色区。理想的结果是只有当鼠标点击区域为蓝色区域时才输出` hello。`

但事实确不是如此，示例Demo。无论点击红色还是蓝色区域均会输出 `hello`，这是怎么回事呢

## 路径
既然 `isPointInPath(x, y)` 的基于路径判断的，那我们就从路径入手。
```javascript
ctx.arc(r, r, r, 0, Math.PI * 1);
ctx.fillStyle = '#2196f3';
ctx.fill();
ctx.arc(r, r, r, Math.PI * 1, Math.PI * 2);
ctx.fillStyle = '#f44336';
ctx.fill();
```

当我们把画图时的 `ctx.beginPath()` 去掉后，发现生成的图形变成一个红色的整圆了：示例Demo。

`beginPath()` 用来重置路径，由于第一个半圆画完路径未重置，第二个半圆就绘制了两条路径。这似乎解释了上个问题的答案。

在 `isInPath(x, y) `函数中，由于路径没有重置，所以最终最终判断的不止是 `ctx.arc(r, r, r, 0, Math.PI * 1)` 这个路径，还有方法外的画红色圆的路径。两个路径加一起自然就是个整圆，所以无论蓝色区还是红色区都会输出。

## 正确结果
```javascript
function isInPath (x, y){
    ctx.beginPath();
    ctx.arc(r, r, r, 0, Math.PI * 1);
    return ctx.isPointInPath(x, y)
}
```

修改 `isInPath` 函数，加入重置路径，结果正确输出
