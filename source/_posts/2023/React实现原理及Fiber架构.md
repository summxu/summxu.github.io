---
title: React实现原理及Fiber架构
date: 2023-05-12 09:36:14
tags:
  - React
  - JavaScript
---

## vdom

**定义**：vdom 就是根据真实dom的属性使用js重新按照原有结构模拟出一份dom对象，dom api 对真实 dom 做增删改。

```js
{
    type: 'div',
    props: {
        id: 'aaa',
        className: ['bbb', 'ccc'],
        onClick: function() {}
    },
    children: []
}
```

有了 vdom 的好处，就没有和 dom 强绑定了，可以跨平台渲染，比如 native、canvas 等等。

但是要让开发去写这样的 vdom 么？

那肯定不行，这样太麻烦了，大家熟悉的是 html 那种方式，所以要引入一些编译的手段。

## dsl 的编译
dsl 是 domain specific language，领域特定语言的意思，html、css 都是 web 领域的 dsl。

直接写 vdom 太麻烦了，所以前端框架都会设计一套 dsl，然后编译成 render function，执行后产生 vdom。

vue 和 react 都是这样：

> jsx -> render function -> vdom

这套 dsl 怎么设计呢？

前端领域大家熟悉的描述 dom 的方式是 html，最好的方式自然是也设计成那样。

所以 vue 的 template，react 的 jsx 就都是这么设计的。

vue 的 template compiler 是自己实现的，而 react 的 jsx 的编译器是 babel 实现的，是两个团队合作的结果。

## 渲染 vdom
渲染 vdom 也就是通过 dom api 增删改 dom。

比如一个 div，那就要 document.createElement 创建元素，然后 setAttribute 设置属性，addEventListener 设置事件监听器。

如果是文本，那就要 document.createTextNode 来创建。

所以说根据 vdom 类型的不同，写个 if else，分别做不同的处理就行了。

没错，不管 vue 还是 react，渲染器里这段 if else 是少不了的：

```js
switch (vdom.tag) {
  case HostComponent:
    // 创建或更新 dom
  case HostText:
    // 创建或更新 dom
  case FunctionComponent: 
    // 创建或更新 dom
  case ClassComponent: 
    // 创建或更新 dom
}
```

react 里是通过 tag 来区分 vdom 类型的，比如 `HostComponent` 就是元素，`HostText` 就是文本，`FunctionComponent`、`ClassComponent` 就分别是函数组件和类组件。

## 状态管理
react **没有响应式系统**，是通过 `setState` 的 api 触发状态更新的，更新以后就**重新渲染整个 vdom**。

> vue 是通过对状态做代理，get 的时候收集以来，然后修改状态的时候就可以触发对应组件的 render 了，不管是子组件、父组件、还是其他位置的组件，只要用到了对应的状态，那就会被作为依赖收集起来，状态变化的时候就可以触发它们的 render，不管是组件是在哪里的。

这就是为什么 react 需要重新渲染整个 vdom，而 vue 不用。

react 的 `setState` 会渲染整个 vdom，而一个应用的所有 vdom 可能是很庞大的，计算量就可能很大。

浏览器里 js 计算时间太长是会阻塞渲染的，会占用每一帧的动画、重绘重排的时间，这样动画就会卡顿。

作为一个有追求的前端框架，动画卡顿肯定是不行的。但是因为 `setState` 的方式只能渲染整个 vdom，所以计算量大是不可避免的。

那能不能把计算量拆分一下，每一帧计算一部分，不要阻塞动画的渲染呢？

顺着这个思路，react 就改造为了 fiber 架构。

## Fiber
### 一：Fiber的概念

React Fiber是react执行渲染时的一种新的调度策略，JavaScript是单线程的，一旦组件开始更新，主线程就一直被React控制，这个时候如果再次执行交互操作，就会卡顿。

  React Fiber就是通过对象记录组件上需要做或者已经完成的更新，一个组件可以对应多个Fiber。

  在render函数中创建的React Element树在第一次渲染的时候会创建一颗结构一模一样的的Fiber节点树。不同的React Element类型对应不同的Fiber节点类型。一个React Element的工作就由它对应的Fiber节点来负责。

  一个React Element可以对应不止一个Fiber，因为Fiber在update的时候，会从原来的Fiber(我们称为current)clone出一个新的Fiber(我们称之为alternate)。俩个Fiber diff出的变化(side effect)记录在alternate上。所以一个组件在更新时最多会有俩个Fiber与其对应，在更新结束后alternate会取代之前的current称为新的current节点。

  React Fiber重构这种方式，渲染过程采用切片的方式，每执行一会儿，就歇一会儿。如果有优先级更高的任务到来以后呢，就会先去执行，降低页面发生卡顿的可能性，使得React对动画等实时性要求较高的场景体验更好。

### 二：什么是Fiber？

  当js在处理大型计算的时候会导致页面出现卡帧的现象，更严重的会出现页面“假死”。所以在这些情况下，必然会导致动画丢帧、不连贯，用户体验就特别差。为了解决这个问题，我们可以将大型的计算拆分成一个个小型计算，然后按照执行顺序异步调用，这样就不会长时间霸占线程，UI也能在俩次小型计算的执行间隙进行更新，从而给与用户及时的反馈，Fiber就是这样做的，并且以一种更高逼格的方式实现了。  
**Driving Idea**  
  如果说v16.0之前的React解决了HOW(如何用最少的DOM操作成本来update视图)的问题，那么这一次Fiber的出现，在这个基础上还解决了WHEN(何时update视图的哪一部分)的问题。  
   **分片优先级！！！**

  基于上述这些原因，Fiber实现了一个虚拟调用栈，并给所有的update进行优先级排序，如下：

```
'use strict';

export type PriorityLevel = 0 | 1 | 2 | 3 | 4 | 5;

module.exports = {
 
NoWork: 0, // No work is pending.
 
SynchronousPriority: 1, // 用于控制文本输入。同步的副作用.

AnimationPriority: 2, //需要在下一帧之前完成.
 
HighPriority: 3, // 需要很快完成的互动才能产生反应.
 
LowPriority: 4, // 数据获取，或更新存储的结果.
 
OffscreenPriority: 5, // 将不可见，但做的工作，以防它成为可见.

};

```

  然后根据这些update的优先级，来决定执行的顺序。  
  我们可以看到动画和页面交互都是优先级比较高的，这也是Fiber能够使得动画、布局和页面交互变得更加的流畅的原因之一。  
  可以把Priority分为同步和异步两个类别，同步优先级的任务会在当前帧完成，包括SynchronousPriority和TaskPriority。异步优先级的任务则可能在接下来的几个帧中被完成，包括HighPriority、LowPriority以及OffscreenPriority。  
  React v16.3.2的优先级，不再这么划分，分为三类：NoWork、sync、async，前两类可以认为是同步任务，需要在当前tick完成，过期时间为null，最后一类异步任务会计算一个。  
  expirationTime，在workLoop中，根据过期时间来判断是否进行下一个分片任务，scheduleWork中更新任务优先级，也就是更新这个expirationTime。至于这个时间怎么计算，可以查看源码。

### 三：Fiber的基本原则：

  更新任务分成俩个阶段，Reconcilition Phase(调和阶段)和Commit Phase(交付阶段)。Reconciliation Phase的任务干的事情是，找出要做的更新工作(Diff Fiber Tree),就是一个计算阶段，计算结果可以被缓存，也就可以被打断；Commit Phase需要提交所有更新并渲染，为了防止页面抖动，被设置为不能打断。  
  **PS：componentWillMount**  
  omponentWillReceiveProps componentWillUpdate 几个生命周期方法，在Reconciliation Phase被调用，有被打断的可能（时间用尽等情况），所以可能被多次调用。其实shouldComponentUpdate 也可能被多次调用，只是它只返回true或者false，没有副作用，可以暂时忽略。

### 四：Fiber的数据结构

  fiber是个链表，有child和sibing属性，指向第一个子节点和相邻的兄弟节点，从而构成fiber tree。return 属性指向其父节点。  
  更新队列，updateQueue，是一个链表，有first和last俩个属性，指向第一个和最后一个update对象。  
  每个fiber有一个属性updateQueue指向其对应的更新队列。  
  每个fiber(当前fiber可以称为current)有一个属性alternate，开始时指向一个自己的clone体，update的变化会先更新到alternate上，当更新完毕，alternate替换current。

### 五：Fiber的执行流程

1.  用户操作引起setState被调用以后，先调用enqueueSetState方法，该方法可以划分成俩个阶段（个人理解），第一阶段Data Preparation，是初始化一些数据结构，比如fiber，updateQueue，update。
2.  新的update会通过insertUpdateIntoQueue方法，根据优先级插入到队列的对应位置，ensureUpdateQueues方法初始化俩个更新队列，queue1和current.updateQueue对应，queue2和current.alternate.updateQueue对应。
3.  第二阶段，Fiber Reconciler，就开始进行任务分片调度，scheduleWork首先更新每个fiber的优先级，这里并没有updatePriority这个方法，但是干了这件事。当fiber.return === null，找到父节点，把所有diff出的变化(side effect)归结到root上。
4.  requestWork，首先把当前的更新添加到schedule list中(addRootToSchedule),然后根据当前是否为异步渲染(isAsync参数)，异步渲染调用。scheduleCallbackWithExpriation方法，下一步高能！！
5.  scheduleCallbackWithExpriation这个方法在不同环境，实现不一样，chrome等浏览器中使用requestIdleCallback API，没有这个API的浏览器中，通过requestAnimationFrame模拟一个requestIdCallback，来在浏览器空闲时，完成下一个分片的工作，注意，这个函数会传入一个expirationTime，超过这个时间活没干完，就放弃了。
6.  执行到performWorkOnRoot，就是fiber文档中提到的Commit Phase和Reconciliation Phase俩阶段。
7.  第一阶段Reconciliation Phase,在workLoop中，通过一个while循环，完成每个分片任务。
8.  performUnitOfWork也可以分成俩阶段，蓝色框表示。beginWork是一个入口函数，根据workInProgress的类型去实例化不同的react element class。workInProgress是通过alternate挂载一些新属性获得的。
9.  实例化不同的react element class时候会调用和will有关的生命周期方法。
10.  completeUnitOfWork是进行一些收尾工作，diff完一个节点以后，更新props和调用生命周期方法等。
11.  然后进入Commit Phase阶段，这个阶段不能被打断。

### 六：Fiber对开发者有什么影响？

1.  componentWillMount,componentWillReceiveProps,componentWillUpdate几个生命周期方法不再安全，由于任务执行过程可以被打断，这几个生命周期可能会执行多次，如果它们包含副作用(比如Ajax)，会有意想不到的bug。React团队提供了替换的生命周期方法。建议如果使用以上方法，尽量使用纯函数，避免以后踩坑。
2.  需要关注react为任务片设置的优先级，特别是页面用动画的情况。
3.  如果一直有更高的级别任务，那么fiber算法会先执行级别更高的任务，执行完毕后再通过callback回到之前渲染到一半的组件，从头开始渲染。（看起来放弃已经渲染完的生命周期，会有点不合理，反而会增加渲染时长，但是react确实是这么干的）