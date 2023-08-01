---
title: Redux源码全面解析
date: 2022-07-26 09:36:37
tags:
  - JavaScript
---

Redux使用中的几个点：

1.  Redux三大设计原则
2.  Create Store
3.  Redux middleware
4.  combineReducer
5.  Provider与Connect
6.  Redux流程梳理
7.  Redux设计特点

#### 1\. 单一数据源

在传统的 MVC 架构中，可以根据需要创建无数个 Model，而 Model 之间可以互相监听、触发事件甚至循环或嵌套触发事件，这些在 Redux 中都是不允许的。因为在 Redux 的思想里，一个应用永远只有唯一的数据源。实际上，使用单一数据源的好处在于整个应用状态都保存在一个对象中，这样随时可以提取出整个应用的状态进行持久化（比如实现一个针对整个应用的即时保存功能）。此外，这样的设计也为服务端渲染提供了可能。

#### 2\. 状态是只读的

在 Redux 中，并不会自己用代码来定义一个 store。取而代之的是，定义一个 reducer，它的功能是根据当前触发的 action 对当前应用的状态（state）进行迭代，这里并没有直接修改应用的状态，而是返回了一份全新的状态。

Redux 提供的 createStore 方法会根据 reducer 生成 store。最后，可以利用 store. dispatch方法来达到修改状态的目的。

#### 3.状态修改均由纯函数完成

在 Redux 里，通过定义 reducer 来确定状态的修改，而每一个 reducer 都是纯函数，这意味着它没有副作用，即接受一定的输入，必定会得到一定的输出。

这样设计的好处不仅在于 reducer 里对状态的修改变得简单、纯粹、可测试，更有意思的是，Redux 利用每次新返回的状态生成酷炫的时间旅行（time travel）调试方式，让跟踪每一次因为触发 action 而改变状态的结果成为了可能。

## 2.Create Store

从store的诞生开始说起。create store函数API文档如下：

```JavaScript
createStore(reducer, [initialState], enhancer)
```

可以看出，它接受三个参数：reducer、initialState 和 enhancer 。Store enhancer 是一个组合 store creator 的高阶函数，返回一个新的强化过的 store creator。这与 middleware 相似，它也允许你通过复合函数改变 store 接口。

再来看看他的返回值：

```JavaScript
{
    dispatch: f (action),
    getState: f (),
    replaceReducer: f (nextReducer),
    subscribe: f (listener),
    Symbol(observable): f ()    
}
```

store的返回值就是一个普通对象，里面有几个常用的方法：

-   dispatch：就是最常用的dispatch方法，派发action。
-   getState：通过该方法，可以拿到当前状态树state。
-   replaceReducer：这个方法主要用于 reducer 的热替换，下面介绍该方法。
-   subscribe：添加一个变化监听器。每当 dispatch（action）的时候就会执行，state 树中的一部分可能已经变化。
-   observable：观察者模式，用于处理订阅关系。

这里挑几个方法介绍：

### getState

在完成基本的参数校验之后，在 createStore 中声明如下变量及 getState 方法：

```JavaScript
var currentReducer = reducer
var currentState = initialState
var listeners = [] // 当前监听 store 变化的监听器
var isDispatching = false // 某个 action 是否处于分发的处理过程中
/**
* Reads the state tree managed by the store.
 *
* @returns {any} The current state tree of your application.
 */
function getState() {
 return currentState
}
```

getState方法就是简单返回当前state，如果state没有被reducer处理过，他就是initialState。

### subscribe

在 getState 之后，定义了 store 的另一个方法 subscribe：

```JavaScript
function subscribe(listener) {
 listeners.push(listener)
 var isSubscribed = true
 return function unsubscribe() {
 if (!isSubscribed) {
 return
 }
 isSubscribed = false
 var index = listeners.indexOf(listener)
 listeners.splice(index, 1)
 }
}
```

Store 允许使用`store.subscribe`方法设置监听函数，一旦 State 发生变化，就自动执行这个函数。

显然，只要把 View 的更新函数（对于 React 项目，就是组件的`render`方法或`setState`方法）放入`listen`，就会实现 View 的自动渲染。你可能会感到奇怪，好像在 Redux 应用中并没有使用 store.subscribe 方法？事实上，

React Redux 中的 connect 方法隐式地帮完成了这个工作。

`store.subscribe`方法返回一个函数，调用这个函数就可以解除监听。

### dispatch

dispatch是redux的核心方法：

```JavaScript
function dispatch(action) {
    if (!isPlainObject(action)) {
        throw new Error(
            'Actions must be plain objects. ' +
            'Use custom middleware for async actions.'
        )
    }
    if (typeof action.type === 'undefined') {
        throw new Error(
            'Actions may not have an undefined "type" property. ' +
            'Have you misspelled a constant?'
            )
        }
    if (isDispatching) {
        throw new Error('Reducers may not dispatch actions.')
    }
    try {
        isDispatching = true
        currentState = currentReducer(currentState, action)
    } finally {
        isDispatching = false
    }
    listeners.slice().forEach(listener => listener())
    return action
}
```

判断当前是否处于某个 action 的分发过程中，这个检查主要是为了避免在 reducer 中分发 action 的情况，因为这样做可能导致分发死循环，同时也增加了数据流动的复杂度。

确认当前不属于分发过程中后，先设定标志位，然后将当前的状态和 action 传给当前的reducer，用于生成最新的 state。这看起来一点都不复杂，这也是反复强调的 reducer 工作过程——纯函数、接受状态和 action 作为参数，返回一个新的状态。

在得到新的状态后，依次调用所有的监听器，通知状态的变更。需要注意的是，在通知监听器变更发生时，并没有将最新的状态作为参数传递给这些监听器。这是因为在监听器中，可以直接调用 store.getState() 方法拿到最新的状态。

最终，处理之后的 action 会被 dispatch 方法返回。

### replaceReducer

```JavaScript
function replaceReducer(nextReducer) {
    if (typeof nextReducer !== 'function') {
      throw new Error('Expected the nextReducer to be a function.');
    }
    currentReducer = nextReducer;
    dispatch({ type: ActionTypes.INIT });
  }
```

这是为了拿到所有 reducer 中的初始状态（你是否还记得在定义 reducer 时，第一个参数为previousState，如果该参数为空，提供默认的 initialState）。只有所有的初始状态都成功获取后，Redux 应用才能有条不紊地开始运作。

## 3.Redux middleware

> It provides a third-party extension point between dispatching an action, and the moment it reachesthe reducer

它提供了一个分类处理 action 的机会。在middleware 中，你可以检阅每一个流过的 action，挑选出特定类型的action 进行相应操作，给你一次改变 action 的机会。

常规的同步数据流模式的流程图如下：![f2BVgyd4bOqmbmPx.png](https://sslstatic.ktanx.com/images/release/201811/f2BVgyd4bOqmbmPx.png)不同业务需求下，比如执行action之前和之后都要打log；action触发一个异步的请求，请求回来之后渲染view等。需要为这一类的action添加公共的方法或者处理，使用redux middleware流程图如下：![Tr2gHBdB9tYoeS97.png](https://sslstatic.ktanx.com/images/release/201811/Tr2gHBdB9tYoeS97.png)每一个 middleware 处理一个相对独立的业务需求，通过串联不同的 middleware 实现变化多样的功能。比如上面的业务，把处理log的代码封装成一个middleware，处理异步的也是一个middleware，两者串联，却又相互独立。

使用middleware之后，action触发的dispatch并不是原来的dispatch，而是经过封装的new dispatch，在这个new dispatch中，按照顺序依次执行每个middleware，最后调用原生的dispatch。

来看下logger middleware如何实现的：

```JavaScript
export default store => next => action => {
    console.log('dispatch:', action); 
    next(action);
    console.log('finish:', action);
 }
```

这里代码十分简洁，就是在next调用下一个middleware之前和之后，分别打印两次。

Redux 提供了 applyMiddleware 方法来加载 middleware，该方法的源码如下：

```JavaScript
import compose from './compose';
export default function applyMiddleware(...middlewares) {
    return function (next) {
        return function (reducer, initialState) {
            let store = next(reducer, initialState);
            let dispatch = store.dispatch;
            let chain = [];
            var middlewareAPI = {
                getState: store.getState,
                dispatch: (action) => dispatch(action),
            };
            chain = middlewares.map(middleware => middleware(middlewareAPI));
            dispatch = compose(...chain)(store.dispatch);
            return {
                ...store,
                dispatch,
            };
        }
    }
}
```

其中compose源码如下：

```JavaScript
function compose(...funcs) {
    return arg => funcs.reduceRight((composed, f) => f(composed), arg);
}
```

使用的时候，如下：

```JavaScript
const newStore = applyMiddleware([mid1, mid2, mid3, ...])(createStore)(reducer, initialState);
```

ok，相关源码已就位，来详细解析一波。

**函数式编程思想设计** ：middleware 的设计有点特殊，是一个层层包裹的匿名函数，这其实是函数式编程中的currying，它是一种使用匿名单参数函数来实现多参数函数的方法。applyMiddleware 会对 logger 这个middleware 进行层层调用，动态地将 store 和 next 参数赋值。currying 的 middleware 结构的好处主要有以下两点。

-   易串联：currying 函数具有延迟执行的特性，通过不断 currying 形成的 middleware 可以累积参数，再配合组合（compose）的方式，很容易形成 pipeline 来处理数据流。
-    共享 store: 在 applyMiddleware 执行的过程中，store 还是旧的，但是因为闭包的存在，applyMiddleware 完成后，所有的 middleware 内部拿到的 store 是最新且相同的。

**给 middleware 分发 store**：newStore创建完成之后，applyMiddleware 方法陆续获得了3个参数，第一个是 middlewares 数组\[mid1, mid2, mid3, ...\]，第二个是 Redux 原生的 createStore ，最后一个是 reducer。然后，可以看到 applyMiddleware 利用 createStore 和 reducer 创建了一个 store。而 store 的 getState方法和 dispatch 方法又分别被直接和间接地赋值给 middlewareAPI 变量 store：

```JavaScript
const middlewareAPI = {
 getState: store.getState,
 dispatch: (action) => dispatch(action),
};
chain = middlewares.map(middleware => middleware(middlewareAPI));
```

然后，让每个 middleware 带着 middlewareAPI 这个参数分别执行一遍。执行完后，获得 chain数组 \[f1, f2, ... , fx, ..., fn\]，它保存的对象是第二个箭头函数返回的匿名函数。因为是闭包，每个匿名函数都可以访问相同的 store，即 middlewareAPI。

> middlewareAPI 中的 dispatch 为什么要用匿名函数包裹呢？
> 
> 用 applyMiddleware 是为了改造 dispatch，所以 applyMiddleware 执行完后，dispatch 是变化了的，而 middlewareAPI 是 applyMiddleware 执行中分发到各个 middleware 的，所以必须用匿名函数包裹 dispatch，这样只要 dispatch 更新了，middlewareAPI 中的 dispatch 应用也会发生变化。

**组合串联 middleware**：这一层只有一行代码，却是 applyMiddleware 精华之所在`dispatch = compose(...chain)(store.dispatch);`，其中 compose 是函数式编程中的组合，它将 chain 中的所有匿名函数 \[f1, f2, ... , fx, ..., fn\]组装成一个新的函数，即新的 dispatch。当新 dispatch 执行时，\[f1, f2, ... , fx, ..., fn\]，从右到左依次执行。

compose(...funcs) 返回的是一个匿名函数，其中 funcs 就是 chain 数组。当调用 reduceRight时，依次从 funcs 数组的右端取一个函数 fx 拿来执行，fx 的参数 composed 就是前一次 fx+1 执行的结果，而第一次执行的 fn（n 代表 chain 的长度）的参数 arg 就是 store.dispatch。所以，当 compose 执行完后，得到的 dispatch 是这样的，假设 n = 3：

```JavaScript
dispatch = f1(f2(f3(store.dispatch))));
```

这时调用新 dispatch，每一个 middleware 就依次执行了。

**在 middleware 中调用 dispatch 会发生什么**：经过 compose 后，所有的 middleware 算是串联起来了。可是还有一个问题，在分发 store 时，提到过每个 middleware 都可以访问 store，即 middlewareAPI 这个变量，也可以拿到 store 的dispatch 属性。那么，在 middleware 中调用 store.dispatch() 会发生什么，和调用 next() 有区别吗？现在来说明两者的不同：

```JavaScript
const logger = store => next => action => {
 console.log('dispatch:', action);
 next(action);
 console.log('finish:', action);
};
const logger = store => next => action => {
 console.log('dispatch:', action);
 store.dispatch(action);
 console.log('finish:', action);
};
```

在分发 store 时解释过，middleware 中 store 的 dispatch 通过匿名函数的方式和最终compose 结束后的新 dispatch 保持一致，所以，在 middleware 中调用 store.dispatch() 和在其他任何地方调用的效果一样。而在 middleware 中调用 next()，效果是进入下一个 middleware，下图就是redux middleware最著名的洋葱模型图。![IArjf9kjKecPhnFF.png](https://sslstatic.ktanx.com/images/release/201811/IArjf9kjKecPhnFF.png)

## 4.combineReducer

如果一个项目过大，通常按模块来写reducer，但是redux create store只接受一个reducer参数，所以需要合并reducer。这里就用到了redux提供的`combineReducer`辅助函数：

```JavaScript
combineReducers({
      layout,
      home,
      ...asyncReducers
  })
```

这个函数用起来很简单，就是传入一个对象，key是模块reducer对应的名字， 值是对应reducer。值是一个function，相当于是一个新的reducer，源码如下：

```JavaScript
export default function combineReducers(reducers) {
  var reducerKeys = Object.keys(reducers)
  var finalReducers = {}
  for (var i = 0; i < reducerKeys.length; i++) {
    var key = reducerKeys[i]
    if (process.env.NODE_ENV !== 'production') {
      if (typeof reducers[key] === 'undefined') {
        warning(`No reducer provided for key "${key}"`)
      }
    }
    if (typeof reducers[key] === 'function') {
      finalReducers[key] = reducers[key]
    }
  }
  var finalReducerKeys = Object.keys(finalReducers)
  if (process.env.NODE_ENV !== 'production') {
    var unexpectedKeyCache = {}
  }
  var sanityError
  try {
    assertReducerSanity(finalReducers)
  } catch (e) {
    sanityError = e
  }
  return function combination(state = {}, action) {
    if (sanityError) {
      throw sanityError
    }
    if (process.env.NODE_ENV !== 'production') {
      var warningMessage = getUnexpectedStateShapeWarningMessage(state, finalReducers, action, unexpectedKeyCache)
      if (warningMessage) {
        warning(warningMessage)
      }
    }
    var hasChanged = false
    var nextState = {}
    for (var i = 0; i < finalReducerKeys.length; i++) {
      var key = finalReducerKeys[i]
      var reducer = finalReducers[key]
      var previousStateForKey = state[key]
      var nextStateForKey = reducer(previousStateForKey, action)
      if (typeof nextStateForKey === 'undefined') {
        var errorMessage = getUndefinedStateErrorMessage(key, action)
        throw new Error(errorMessage)
      }
      nextState[key] = nextStateForKey
      hasChanged = hasChanged || nextStateForKey !== previousStateForKey
    }
    return hasChanged ? nextState : state
  }
}
```

源码不是很多，除去一些验证代码，剩下的就是说：return一个function，暂时称呼他combination，就相当于是与一个总的reducer，每次action都会走到combination中，combination会遍历输入的reducer，将action放到每个reducer中执行一下，计算出返回结果就是nextState，nextState于previousState如果!==说明改变了，返回nextState，否则返回执行之前的state。

这也解释了不同模块actionType如果相同的话，两个模块的reducer都会走一遍的问题，在actionType名称前面加上模块前缀即可解决问题。

## 5\. Provider与Connect

Provider与Connet组件都是React-Redux提供的核心组件，两者看起来功能一样，都是帮助容器组件获取store中的数据，但是原理与功能却不同。

### Provider

Provider组件在所有组件的最外层，其接受store作为参数，将store里的state使用context属性向下传递。部分源码：

```JavaScript
export default class Provider extends Component {
 getChildContext() {
 return { store: this.store }
 }
 constructor(props, context) {
 super(props, context)
 this.store = props.store
 }
 render() {
 const { children } = this.props
 return Children.only(children)
 }
}
```

利用context这个属性，Provider所有子组件均可以拿到这个属性。

### Connect

connect实现的功能是将需要关联store的组件和store的dispatch等数据混合到一块，这块就是一个高阶组件典型的应用：

```JavaScript
import hoistStatics from 'hoist-non-react-statics'
export default function connect(mapStateToProps, mapDispatchToProps, mergeProps, options = {}) {
 // ...
 return function wrapWithConnect(WrappedComponent) {
 // ...
 class Connect extends Component {
 // ...
 render() {
 // ...
 if (withRef) {
 this.renderedElement = createElement(WrappedComponent, {
 ...this.mergedProps,
 ref: 'wrappedInstance'
 })
 } else {
 this.renderedElement = createElement(WrappedComponent,
 this.mergedProps
 )
 }
 return this.renderedElement
 }
 }
 // ...
 return hoistStatcis(Connect, WrappedComponent);
 }
}
```

还是先从他的四个参数说起：

#### 1.mapStateToProps

connect 的第一个参数定义了需要从 Redux 状态树中提取哪些部分当作 props 传给当前组件。一般来说，这也是使用 connect 时经常传入的参数。事实上，如果不传入这个参数，React 组件将永远不会和 Redux 的状态树产生任何关系。具体在源代码中的表现为：

```JavaScript
export default function connect(mapStateToProps, mapDispatchToProps, mergeProps, options = {}) {
 const shouldSubscribe = Boolean(mapStateToProps)
 // ...
 class Connect extends Component {
 // ...
 trySubscribe() {
 if (shouldSubscribe && !this.unsubscribe) {
 this.unsubscribe = this.store.subscribe(this.handleChange.bind(this))
 this.handleChange()
 }
 }
 // ...
 }
}
```

`mapStateToProps`会订阅 Store，每当`state`更新的时候，就会自动执行，重新计算 UI 组件的参数，从而触发 UI 组件的重新渲染。

`mapStateToProps`的第一个参数总是`state`对象，还可以使用第二个参数，代表容器组件的`props`对象。

这块的源码相对较简单：

```JavaScript
const mapState = mapStateToProps || defaultMapStateToProps 
class Connect extends Component { 
    computeStateProps(store, props) {
        if (!this.finalMapStateToProps) {
          return this.configureFinalMapState(store, props)
        }
        const state = store.getState()
        const stateProps = this.doStatePropsDependOnOwnProps ?
          this.finalMapStateToProps(state, props) :
          this.finalMapStateToProps(state)
        if (process.env.NODE_ENV !== 'production') {
          checkStateShape(stateProps, 'mapStateToProps')
        }
        return stateProps
      }
      configureFinalMapState(store, props) {
        const mappedState = mapState(store.getState(), props)
        const isFactory = typeof mappedState === 'function'
        this.finalMapStateToProps = isFactory ? mappedState : mapState
        this.doStatePropsDependOnOwnProps = this.finalMapStateToProps.length !== 1
        if (isFactory) {
          return this.computeStateProps(store, props)
        }
        if (process.env.NODE_ENV !== 'production') {
          checkStateShape(mappedState, 'mapStateToProps')
        }
        return mappedState
      }
}
```

这块原理很简单，进行一些参数校验，判断第一个参数mapStateToProps返回值是否为function，如果是递归调用，不是的话算出返回值。如果没传这个参数，默认给{}。

> 可能会疑惑为什么传给 connect 的第一个参数本身是一个函数，react-redux 还允许这个函数的返回值也是一个函数呢？简单地说，这样设计可以允许在 connect 的第一个参数里利用函数闭包进行一些复杂计算的缓存，从而实现效率优化的目的

当使用的时候：

```JavaScript
const mapStateToProps = (state, props) => ({
    home: state.home,
    layout: state.layout
});
```

使用`ownProps`作为参数后，如果容器组件的参数发生变化，也会引发 UI 组件重新渲染

#### 2.mapDispatchToProps

人如其名，它接受 store 的 dispatch 作为第一个参数，同时接受 this.props 作为可选的第二个参数。利用这个方法，可以在 connect 中方便地将 actionCreator 与 dispatch 绑定在一起（利用 bindActionCreators 方法），最终绑定好的方法也会作为 props 传给当前组件。这块的源码与mapStateToProps一样，就不贴了。

bindActionCreator

```JavaScript
function bindActionCreator(actionCreator, dispatch) {
  return (...args) => dispatch(actionCreator(...args))
}
```

#### 3.mergeProps

前两个参数返回的对象，都要跟组件自身的props merge一下，形成一个新的对象赋值给对应组件，可以在这一步做一些处理，这个参数就是干这个的，该参数签名：

```JavaScript
mergeProps(stateProps, dispatchProps, ownProps): props
```

默认情况如果没传该参数，返回`Object.assign(ownProps, stateProps, dispatchProps)`。

#### 4.options

如果指定这个参数，可以定制 connector 的行为。

-   \[`pure = true`\] _(Boolean)_: 如果为 true，connector 将执行 `shouldComponentUpdate` 并且浅对比 `mergeProps` 的结果，避免不必要的更新，前提是当前组件是一个“纯”组件，它不依赖于任何的输入或 state 而只依赖于 props 和 Redux store 的 state。\*默认值为 true。\*
-   \[`withRef = false`\] _(Boolean)_: 如果为 true，connector 会保存一个对被包装组件实例的引用，该引用通过 `getWrappedInstance()` 方法获得。\*默认值为 false。\*

这个connect组件还干了一件事，状态缓存判断。当store变了的时候，前后状态判断，如果状态不等，更新组件，并且完成事件分发。

## 6\. Redux流程梳理

上面讲了大量的函数源码，这么些函数之间的关系：![4g40yEqEfjeRqxs6.png](https://sslstatic.ktanx.com/images/release/201811/4g40yEqEfjeRqxs6.png)初始化阶段：

1.  createStore创建一个store对象
2.  将store对象通过参数给Provider组件
3.  Provider组件将store通过context向子组件传递
4.  Connect组件通过context获取到store，存入自己的state
5.  componentDidMount里面订阅store.subscribe事件

更新数据阶段：

1.  用户事件触发
2.  actionCreator生成action交给dispatch
3.  实际上交给了封装后的中间层（compose(applyMiddleware(...))）
4.  请求依次通过每个中间件，中间件通过next进行下一步
5.  最后一个中间件将action交给store.dispatch
6.  dispatch内部将action交给reducer执行
7.  combineReducer将每个子reducer执行一遍算出新的state
8.  dispatch内部调用所有订阅事件
9.  Connect组件handleChange事件触发判断新state和旧state是否===
10.  并且判断新的state是否与mapStateToProps shallowEqual
11.  不等则setState触发更新

## 7.Redux设计技巧

1.  匿名函数&&闭包使用
    
    redux核心函数大量使用了匿名函数和闭包来实现数据共享和状态同步。
    
2.  函数柯里化使用
    
    使用函数柯里化s实现参数复用，本质上是降低通用性，提高适用性。
    
3.  核心状态读取是拷贝而不是地址
    
    对于state这种核心状态使用getState()计算出新的state，而不是直接返回一个state对象。
    
4.  观察者订阅者是核心实现
    
    使用观察者订阅者模式实现数据响应。
    
5.  context这个api的使用
    
    平时开发不常接触的api实现Provider与Connect通信。