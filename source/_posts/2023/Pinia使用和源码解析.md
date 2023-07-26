---
title: Pinia使用和源码解析
date: 2023-07-26 09:36:37
tags:
  - Vue
  - JavaScript
---
## 创建pinia

基本使用

```TypeScript
const pinia = createPinia()
const app = createApp(App)

app.use(pinia)
app.mount('#app')
//vue2 使用
import { createPinia, PiniaVuePlugin } from 'pinia'

Vue.use(PiniaVuePlugin)
const pinia = createPinia()

new Vue({
  el: '#app', 
  pinia,
})

```

## createPinia

-   将pinia绑定到实例上
-   创建一个scope用来保存之后创建的每一个store的state，也可以直接通过pinia.state.value\[storeId\]直接设置state

```TypeScript
//ceatePinia.ts
export function createPinia(): Pinia {
  // 创建个scope effct来单独管理state
  const scope = effectScope(true)
  // 通过pinia.state.value[storeId]会保存之后创建的store的state
  const state = scope.run<Ref<Record<string, StateTree>>>(() =>
    ref<Record<string, StateTree>>({})
  )!
  
  // 插件相关
  let _p: Pinia['_p'] = []
  // plugins added before calling app.use(pinia)
  let toBeInstalled: PiniaPlugin[] = []
  
  // markRaw标记pinia不可被代理,避免存在用户将其响应式化影响性能的情况
  const pinia: Pinia = markRaw({
    install(app: App) {
  // 设置当前活跃的pinia，方便其他地方获取
      setActivePinia(pinia)
      if (!isVue2) {
        pinia._a = app
        // 内部通过inject获取pinia,因为piniaSymbol没有导出，所以开发者无法通过inject获取pinia
        app.provide(piniaSymbol, pinia)
        // 通过app.config.globalProperties将pinia挂载到组件实例上
        app.config.globalProperties.$pinia = pinia
        if (USE_DEVTOOLS) {
          registerPiniaDevtools(app, pinia)
        }
        toBeInstalled.forEach((plugin) => _p.push(plugin))
        toBeInstalled = []
      }
    },
    // pinia插件相关，暂不分析
    use(plugin) {
      if (!this._a && !isVue2) {
        toBeInstalled.push(plugin)
      } else {
        _p.push(plugin)
      }
      return this
    },

    _p,
    // it's actually undefined here
    // @ts-expect-error
    _a: null,
    _e: scope,
    // 每个创建的store都会放在这个Map里
    _s: new Map<string, StoreGeneric>(),
    state,
  })

  //...
  return pinia
}
```

## 设置、获取pinia

-   pinia会被保存到一个全局变量上，内部可以通过setActivePinia、getActivePinia快速获取到pinia，然后使用上一步挂在pinia上的属性或方法

```TypeScript
//rootStore.ts

export let activePinia: Pinia | undefined

export const setActivePinia = (pinia: Pinia | undefined) =>
  (activePinia = pinia)
  
// 如果在vue组件中，通过inject获取（在createPinia中导出）、否则直接取全局变量 
export const getActivePinia = () =>
  (getCurrentInstance() && inject(piniaSymbol)) || activePinia

```

## 兼容vue2

-   看过vuex的这段应该很熟悉
-   通过Object.defineProperty仿造了个Provide功能
-   通过mixins将pinia注入到每一个组件中

PiniaVuePlugin

```TypeScript
//vue2-plugin.ts
export const PiniaVuePlugin: Plugin = function (_Vue) {
  _Vue.mixin({
    beforeCreate() {
      const options = this.$options
      if (options.pinia) {
      // 获取注册到根组件上的pinia
        const pinia = options.pinia as Pinia
        // 通过Object.defineProperty实现的hack版provid、inject...
        if (!(this as any)._provided) {
          const provideCache = {}
          Object.defineProperty(this, '_provided', {
            get: () => provideCache,
            set: (v) => Object.assign(provideCache, v),
          })
        }
        ;(this as any)._provided[piniaSymbol as any] = pinia

        if (!this.$pinia) {
          this.$pinia = pinia
        }

        pinia._a = this as any
        if (IS_CLIENT) {
          // this allows calling useStore() outside of a component setup after
          // installing pinia's plugin
          setActivePinia(pinia)
        }
        if (USE_DEVTOOLS) {
          registerPiniaDevtools(pinia._a, pinia)
        }
      } else if (!this.$pinia && options.parent && options.parent.$pinia) {
        this.$pinia = options.parent.$pinia
      }
    },
    destroyed() {
      delete this._pStores
    },
  })
}
```

## 创建store

创建store的三种方式

```TypeScript
const useUserStore = defineStore('counter', {
  state: () => ({
    count: 0
  }),
  actions: {
    increment() {
      this.count++
    }
  }
})

// or
const useUserStore = defineStore({
  id: 'counter',
  state: () => ({
    count: 0
  }),
  actions: {
    increment() {
      this.count++
    }
  }
})

// or
const useUserStore = defineStore('counter', () => {
  const count = ref(0)

  function increment() {
    count.value++
  }
  return { count, increment }
})
```

## defineStore

-   主要根据defineStore的不同方式选择调用函数式createSetupStore还是选项式createOptionsStore
-   创建好的store会根据defineStore时传入的id，挂载到pinia.\_s这个map上

```TypeScript
export function defineStore(
  // TODO: add proper types from above
  idOrOptions: any,
  setup?: any,
  setupOptions?: any
): StoreDefinition {
  let id: string
  let options
  
  const isSetupStore = typeof setup === 'function'
  
  // 抹平不同格式参数
  if (typeof idOrOptions === 'string') {
    id = idOrOptions
    options = isSetupStore ? setupOptions : setup
  } else {
    options = idOrOptions
    id = idOrOptions.id
  }
  // 最终交给开发者获取store的方法
  function useStore(pinia?: Pinia | null, hot?: StoreGeneric): StoreGeneric {
  
    const currentInstance = getCurrentInstance()
    
    // 如果存在currentInstance说明在vue组件中，通过inject获取到pinia
    pinia =
      (__TEST__ && activePinia && activePinia._testing ? null : pinia) ||
      (currentInstance && inject(piniaSymbol, null))
    
      //设置当前活跃的pinia对象，如果存在多个pinia对象，方便快速获取当前pinia对象
    if (pinia) setActivePinia(pinia)

    if (__DEV__ && !activePinia) {
      throw new Error(
        `[🍍]: getActivePinia was called with no active Pinia. Did you forget to install pinia?\n` +
          `\tconst pinia = createPinia()\n` +
          `\tapp.use(pinia)\n` +
          `This will fail in production.`
      )
    }
    // 从rootStore文件中取的全局变量
    pinia = activePinia!
    // 如果没有创建过id对应的store,则会调用createSetupStore或createOptionsStore进行创建
    if (!pinia._s.has(id)) {
      // 区分第二个参数是函数还是options对象
      if (isSetupStore) {
        createSetupStore(id, setup, options, pinia)
      } else {
        createOptionsStore(id, options as any, pinia)
      }

      /* istanbul ignore else */
      if (__DEV__) {
        // @ts-expect-error: not the right inferred type
        useStore._pinia = pinia
      }
    }
    // 获取上面创建好的store，pinia._s是一个Map结构
    const store: StoreGeneric = pinia._s.get(id)!
    
    // 热更新相关,重新创建更新store
    if (__DEV__ && hot) {
      const hotId = '__hot:' + id
      const newStore = isSetupStore
        ? createSetupStore(hotId, setup, options, pinia, true)
        : createOptionsStore(hotId, assign({}, options) as any, pinia, true)

      hot._hotUpdate(newStore)

      // cleanup the state properties and the store from the cache
      delete pinia.state.value[hotId]
      pinia._s.delete(hotId)
    }

    // 往当前组件实例上缓存store，主要是给devtools使用
    if (
      __DEV__ &&
      IS_CLIENT &&
      currentInstance &&
      currentInstance.proxy &&
      // avoid adding stores that are just built for hot module replacement
      !hot
    ) {
      const vm = currentInstance.proxy
      const cache = '_pStores' in vm ? vm._pStores! : (vm._pStores = {})
      cache[id] = store
    }

    // StoreGeneric cannot be casted towards Store
    return store as any
  }

  useStore.$id = id

  return useStore
}
```

## createSetupStore

如果defineStore传入的是一个setup函数，则会调用此方法创建store

-   ### 响应式处理store
    
    -   store -> reactive(store)
    -   pinia.state.value\[storeId\]初始化，state会往pinia上存一份和在store上存一份
    -   将store注册到pinia.\_s这个Map上

```TypeScript
function createSetupStore(
  $id: Id,
  setup: () => SS,
  options:
    | DefineSetupStoreOptions<Id, S, G, A>
    | DefineStoreOptions<Id, S, G, A> = {},
  pinia: Pinia,
  hot?: boolean,
  isOptionsStore?: boolean
){
let scope!: EffectScope  //为setup函数返回的内容单独建立一个scope
if (__DEV__ && !pinia._e.active) { //pinia._e为创建pinia时建立的scope
    throw new Error('Pinia destroyed')
  }

const initialState = pinia.state.value[$id] as UnwrapRef<S> | undefined
// 如果pinia.state.value[storeId]未初始化，进行初始化
if (!isOptionsStore && !initialState && (!__DEV__ || !hot)) {
  if (isVue2) {
    set(pinia.state.value, $id, {})
  } else {
    pinia.state.value[$id] = {}
  }
}
//...

// 对外暴露的store内容
const partialStore = {
 _p: pinia,
 // _s: scope,
 $id,
 // 调用$onAction会将回调加入到actionSubscriptions中,wrapAction内会触发回调
 $onAction: addSubscription.bind(null, actionSubscriptions),
 $patch,
 $reset,
 $subscribe(callback, options = {}) {
   const removeSubscription = addSubscription(
     subscriptions,
     callback,
     options.detached,
     () => stopWatcher()
   )
   // 监听state直接修改
   const stopWatcher = scope.run(() =>
     watch(
       () => pinia.state.value[$id] as UnwrapRef<S>,
       (state) => {
         // flush默认为'pre',而在调用$patch时isListening会被设置为false,所以不会触发$patch修改state的监听回调
         if (options.flush === 'sync' ? isSyncListening : isListening) {
           callback(
             {
               storeId: $id,
               type: MutationType.direct,
               events: debuggerEvents as DebuggerEvent,
             },
             state
           )
         }
       },
       assign({}, $subscribeOptions, options)
     )
   )!

   return removeSubscription
 },
 $dispose,
} as _StoreWithState<Id, S, G, A>

// 返回的store是个reactive对象
const store: Store<Id, S, G, A> = reactive(
  __DEV__ || USE_DEVTOOLS
    ? assign(
        {
          _hmrPayload,
          _customProperties: markRaw(new Set<string>()), // devtools custom properties
        },
        partialStore
        // must be added later
        // setupStore
      )
    : partialStore
) as unknown as Store<Id, S, G, A>

// 将store注册到pinia上
pinia._s.set($id, store)

}
```

-   ### 处理setup返回的内容
    

```TypeScript
 function createSetupStore(
  $id: Id,
  setup: () => SS,
  options:
    | DefineSetupStoreOptions<Id, S, G, A>
    | DefineStoreOptions<Id, S, G, A> = {},
  pinia: Pinia,
  hot?: boolean,
  isOptionsStore?: boolean
){
 // ...
 
 // 将defineStore使用者定义的返回内容放进scope中,在组件卸载时回收effect
 const setupStore = pinia._e.run(() => {
   scope = effectScope()
   return scope.run(() => setup())
 })!
 // ...
}
```

-   ### 同步setup返回内容和pinia.state
    
    -   因为state会在store上存一份，也会在pinia.state.value\[storeId\]上存一份，所以为了保证两边都是同一个代理对象，需要进行同步
    -   使用者可以直接通过pinia.state.value设置store内容，所以直接设置的内容也需要同步

```TypeScript
 function createSetupStore(
  $id: Id,
  setup: () => SS,
  options:
    | DefineSetupStoreOptions<Id, S, G, A>
    | DefineStoreOptions<Id, S, G, A> = {},
  pinia: Pinia,
  hot?: boolean,
  isOptionsStore?: boolean
){
 // ...
 
// 将defineStore使用者定义的返回内容放进scope中,在组件卸载时回收effect
const setupStore = pinia._e.run(() => {
  scope = effectScope()
  return scope.run(() => setup())
})!

for (const key in setupStore) {
 const prop = setupStore[key]
 // 只处理ref、reactive对象，computed等不处理
 if ((isRef(prop) && !isComputed(prop)) || isReactive(prop)) {
   // mark it as a piece of state to be serialized
   if (__DEV__ && hot) {
     // 热更新相关，忽略
     set(hotState.value, key, toRef(setupStore as any, key))
     // option结构已经在createOptionsStore将其加入pinia
     
   } else if (!isOptionsStore) {// 同步pinia.state -> store
   
     // 将用户可能直接调用pinia.state.value[$id]设置的ref、reactive对象设置到setup返回的结果上,让二者的响应式都代理一个对象
     // 使得store、pinia能同步更改
     if (initialState && shouldHydrate(prop)) {
       if (isRef(prop)) {
         prop.value = initialState[key]
       } else {
         // probably a reactive object, lets recursively assign
         // 同步其他类型
         mergeReactiveObjects(prop, initialState[key])
       }
     }
     // transfer the ref to the pinia state to keep everything in sync
     // 将setup返回的ref、reactive对象同步到pinia.state上，使得store、pinia能同步更改
     if (isVue2) { //同步 store -> pinia.state
       set(pinia.state.value[$id], key, prop)
     } else {
       pinia.state.value[$id][key] = prop
     }
   }
 }
}

// 合并方法
function mergeReactiveObjects<
  T extends Record<any, unknown> | Map<unknown, unknown> | Set<unknown>
>(target: T, patchToApply: _DeepPartial<T>): T {
  // 合并Map类型
  if (target instanceof Map && patchToApply instanceof Map) {
    patchToApply.forEach((value, key) => target.set(key, value))
  }
  // 合并Set类型
  if (target instanceof Set && patchToApply instanceof Set) {
    patchToApply.forEach(target.add, target)
  }

  for (const key in patchToApply) {
    if (!patchToApply.hasOwnProperty(key)) continue
    const subPatch = patchToApply[key]
    const targetValue = target[key]
    // 只有普通对象才会进入递归
    if (
      isPlainObject(targetValue) &&
      isPlainObject(subPatch) &&
      target.hasOwnProperty(key) &&
      !isRef(subPatch) &&
      !isReactive(subPatch)
    ) {
      target[key] = mergeReactiveObjects(targetValue, subPatch)
    } else {
      target[key] = subPatch
    }
  }

  return target
}

```

-   ### 处理action
    
    -   将action替换成wrapAction，wrapAction添加了订阅发布相关的功能

```TypeScript
 function createSetupStore(
  $id: Id,
  setup: () => SS,
  options:
    | DefineSetupStoreOptions<Id, S, G, A>
    | DefineStoreOptions<Id, S, G, A> = {},
  pinia: Pinia,
  hot?: boolean,
  isOptionsStore?: boolean
){
 // ...
 
// 将defineStore使用者定义的返回内容放进scope中,在组件卸载时回收effect
const setupStore = pinia._e.run(() => {
  scope = effectScope()
  return scope.run(() => setup())
})!

for (const key in setupStore) {
 const prop = setupStore[key]
 // 只处理ref、reactive对象，computed等不处理
 if ((isRef(prop) && !isComputed(prop)) || isReactive(prop)) {
 // ...
 } else if (typeof prop === 'function') { 
 
  // 将setup中的方法替换成wrapAction包装的方法，wrapAction在订阅发布章节解析
  
      const actionValue = __DEV__ && hot ? prop : wrapAction(key, prop)
      
      if (isVue2) {
        set(setupStore, key, actionValue)
      } else {
        // @ts-expect-error
        setupStore[key] = actionValue
      }

      /* istanbul ignore else */
      if (__DEV__) {
        _hmrPayload.actions[key] = prop
      }

      // list actions so they can be used in plugins
      // @ts-expect-error
      optionsForPlugin.actions[key] = prop
    }
 
}
}
```

-   ### 处理store合并setup
    

```TypeScript
 function createSetupStore(
  $id: Id,
  setup: () => SS,
  options:
    | DefineSetupStoreOptions<Id, S, G, A>
    | DefineStoreOptions<Id, S, G, A> = {},
  pinia: Pinia,
  hot?: boolean,
  isOptionsStore?: boolean
){
// ...
// 对外暴露的store API
const partialStore = {
//...
}
const store: Store<Id, S, G, A> = reactive(
//...
partialStore
)
// 将defineStore使用者定义的返回内容放进scope中,在组件卸载时回收effect
const setupStore = pinia._e.run(() => {
scope = effectScope()
return scope.run(() => setup())
})!

for (const key in setupStore) {
// ...
}

 // 将最同步后的结果，合并进store上
 if (isVue2) {
   Object.keys(setupStore).forEach((key) => {
     set(store, key, setupStore[key])
   })
 } else {
   // 将store的reactive对象、原始对象都进行合并
   assign(store, setupStore)
   assign(toRaw(store), setupStore)
 }
}
```

## createOptionsStore

当传入的是option配置时，则会调用此方法创建store

-   createSetupStore内部会将option转换成setup方法，然后实际调用createSetupStore进行创建

```TypeScript
function createOptionsStore<
  Id extends string,
  S extends StateTree,
  G extends _GettersTree<S>,
  A extends _ActionsTree
>(
  id: Id,
  options: DefineStoreOptions<Id, S, G, A>,
  pinia: Pinia,
  hot?: boolean
): Store<Id, S, G, A> {

  const { state, actions, getters } = options

  const initialState: StateTree | undefined = pinia.state.value[id]
  
  let store: Store<Id, S, G, A>
  
  // 将options转化成setup函数
  function setup() {
    // 初始化pinia.state
    if (!initialState && (!__DEV__ || !hot)) {
      if (isVue2) {
        set(pinia.state.value, id, state ? state() : {})
      } else {
        pinia.state.value[id] = state ? state() : {}
      }
    }
    
    // 将pinia.state全部转换成ref，和setup中返回ref效果一致
    const localState =
      __DEV__ && hot
        ? // use ref() to unwrap refs inside state TODO: check if this is still necessary
          toRefs(ref(state ? state() : {}).value)
        : toRefs(pinia.state.value[id])
    
    // 最终options store返回的内容格式和setup store返回的内容格式一致
    return assign(
      localState,
      actions,
      // 将 getter 转换成 computed
      Object.keys(getters || {}).reduce((computedGetters, name) => {
        if (__DEV__ && name in localState) {
          console.warn(
            `[🍍]: A getter cannot have the same name as another state property. Rename one of them. Found with "${name}" in store "${id}".`
          )
        }
// getter函数不可代理，及不对computed做额外处理，和setup中一致
        computedGetters[name] = markRaw(
          computed(() => {
            setActivePinia(pinia)
            // it was created just before
            const store = pinia._s.get(id)!

            // allow cross using stores
            if (isVue2 && !store._r) return

            // @ts-expect-error
            // return getters![name].call(context, context)
            return getters![name].call(store, store)
          })
        )
        return computedGetters
      }, {} as Record<string, ComputedRef>)
    )
  }

  // option选项注册的pinia会被转换成setup函数形式
  store = createSetupStore(id, setup, options, pinia, hot, true)

  store.$reset = function $reset() {
    const newState = state ? state() : {}
    // we use a patch to group all changes into one single subscription
    this.$patch(($state) => {
      assign($state, newState)
    })
  }

  return store as any

}
```

## 订阅发布

pinia能够对修改state、action进行监听，内部通过发布订阅模式、watch API实现

-   ## 发布订阅模式
    
    -   将存储订阅器的功能和整个设计解藕，交给外部来传入，好处是能够处理不同类型的发布订阅

```TypeScript
//subscriptions.ts
export function addSubscription<T extends _Method>(
  subscriptions: T[], //外部传入存储器
  callback: T,
  detached?: boolean, // 是否在组件卸载时清除订阅
  onCleanup: () => void = noop
) {
  subscriptions.push(callback)
  // 清除订阅
  const removeSubscription = () => {
    const idx = subscriptions.indexOf(callback)
    if (idx > -1) {
      subscriptions.splice(idx, 1)
      onCleanup()
    }
  }
  // 如果没传detached,则会自动在组件卸载时清除订阅
  if (!detached && getCurrentScope()) {
    onScopeDispose(removeSubscription)
  }

  return removeSubscription
}

export function triggerSubscriptions<T extends _Method>(
  subscriptions: T[],
  ...args: Parameters<T>
) {
  subscriptions.slice().forEach((callback) => {
    callback(...args)
  })
}
```

-   ## 监听action
    

基本使用

```TypeScript
const unsubscribe = someStore.$onAction(
  ({
    name, // action 名称
    store, // store 实例，类似 `someStore`
    args, // 传递给 action 的参数数组
    after, // 在 action 返回或解决后的钩子
    onError, // action 抛出或拒绝的钩子
  }) => {
    // 这将在执行 "store "的 action 之前触发。
    console.log(`xx`)

    // 这将在 action 成功并完全运行后触发。它等待着任何返回的 promise
    after((result) => {
      console.log(
        `Finished "${name}" after ${
          Date.now() - startTime
        }ms.\nResult: ${result}.`
      )
    })

    // 如果 action 抛出或返回一个拒绝的 promise，这将触发
    onError((error) => {
      console.warn(
        `Failed "${name}" after ${Date.now() - startTime}ms.\nError: ${error}.`
      )
    })
  }
)

// 手动删除监听器
unsubscribe()
```

### 通过$onAction添加订阅

```TypeScript
function createSetupStore<
  Id extends string,
  SS extends Record<any, unknown>,
  S extends StateTree,
  G extends Record<string, _Method>,
  A extends _ActionsTree
>(
  $id: Id,
  setup: () => SS,
  options:
    | DefineSetupStoreOptions<Id, S, G, A>
    | DefineStoreOptions<Id, S, G, A> = {},
  pinia: Pinia,
  hot?: boolean,
  isOptionsStore?: boolean
): Store<Id, S, G, A> {
// ...
// 对外暴露的store内容
const partialStore = {
 _p: pinia,
 // _s: scope,
 $id,
 // 调用$onAction会将回调加入到actionSubscriptions中,调用wrapAction内会触发回调
 $onAction: addSubscription.bind(null, actionSubscriptions),
 $patch,
 $reset,
 $subscribe(callback, options = {}) {
 // ...
 },
 $dispose,
} as _StoreWithState<Id, S, G, A>
// ...
}
```

### 通过wrapAction进行发布

-   setup、option中的方法，会被替换成wrapAction
-   当调用wrapAction时，会触发actionSubscriptions、afterCallbackList或onErrorCallbackList中的回调

```TypeScript
function createSetupStore<
  Id extends string,
  SS extends Record<any, unknown>,
  S extends StateTree,
  G extends Record<string, _Method>,
  A extends _ActionsTree
>(
  $id: Id,
  setup: () => SS,
  options:
    | DefineSetupStoreOptions<Id, S, G, A>
    | DefineStoreOptions<Id, S, G, A> = {},
  pinia: Pinia,
  hot?: boolean,
  isOptionsStore?: boolean
): Store<Id, S, G, A> {

// ...

// internal state
let isListening: boolean // 异步监听
let isSyncListening: boolean // 同步监听

let subscriptions: SubscriptionCallback<S>[] = markRaw([])  // 监听state的订阅集合
let actionSubscriptions: StoreOnActionListener<Id, S, G, A>[] = markRaw([]) // 监听action的订阅集合

// ...

// 包装action调用，追加发布订阅功能
function wrapAction(name: string, action: _Method) {
 return function (this: any) {
   // 根据闭包上下文，调用时设置当前活跃的pinia
   setActivePinia(pinia)
   const args = Array.from(arguments)
   // 调用action后回调集合
   const afterCallbackList: Array<(resolvedReturn: any) => any> = []
   const onErrorCallbackList: Array<(error: unknown) => unknown> = []

   function after(callback: _ArrayType<typeof afterCallbackList>) {
     afterCallbackList.push(callback)
   }
   function onError(callback: _ArrayType<typeof onErrorCallbackList>) {
     onErrorCallbackList.push(callback)
   }

   // 触发action时的回调，同时将after等方法通过参数传入
   triggerSubscriptions(actionSubscriptions, {
     args,
     name,
     store,
     after,
     onError,
   })

   let ret: any
   try {
     ret = action.apply(this && this.$id === $id ? this : store, args)
   } catch (error) {
     // 处理同步错误
     triggerSubscriptions(onErrorCallbackList, error)
     throw error
   }
   // 如果是异步方法
   if (ret instanceof Promise) {
     return ret
       .then((value) => {
         // 获取到action调用结果后触发
         triggerSubscriptions(afterCallbackList, value)
         return value
       })
       .catch((error) => {
         triggerSubscriptions(onErrorCallbackList, error)
         return Promise.reject(error)
       })
   }

   // allow the afterCallback to override the return value
   triggerSubscriptions(afterCallbackList, ret)
   return ret
 }
}

}
```

## 监听修改state

修改state有两种方式，一种通过store.state直接修改然后通过watch API进行监听，另一种通过$patch进行[批量修改](https://so.csdn.net/so/search?q=%E6%89%B9%E9%87%8F%E4%BF%AE%E6%94%B9&spm=1001.2101.3001.7020)，通过$patch进行批量修改时，为了只触发一次回调需要手动触发

-   基本使用

```TypeScript
cartStore.$subscribe((mutation, state) => {
  // import { MutationType } from 'pinia'
  mutation.type // 'direct' | 'patch object' | 'patch function'
  // 和 cartStore.$id 一样
  mutation.storeId // 'cart'
  // 只有 mutation.type === 'patch object'的情况下才可用
  mutation.payload // 传递给 cartStore.$patch() 的补丁对象。
  
  // ...
})

```

-   ### 直接修改state
    

$subscribe监听state直接修改，内部通过watch API实现

```TypeScript
function createSetupStore<
  Id extends string,
  SS extends Record<any, unknown>,
  S extends StateTree,
  G extends Record<string, _Method>,
  A extends _ActionsTree
>(
  $id: Id,
  setup: () => SS,
  options:
    | DefineSetupStoreOptions<Id, S, G, A>
    | DefineStoreOptions<Id, S, G, A> = {},
  pinia: Pinia,
  hot?: boolean,
  isOptionsStore?: boolean
): Store<Id, S, G, A> {
// internal state
let isListening: boolean // set to true at the end
let isSyncListening: boolean // set to true at the end

// ...
// 对外暴露的store内容
const partialStore = {
 _p: pinia,
 // _s: scope,
 $id,
 // 调用$onAction会将回调加入到actionSubscriptions中,调用wrapAction内会触发回调
 $onAction: addSubscription.bind(null, actionSubscriptions),
 $patch,
 $reset,
 $subscribe(callback, options = {}) {
    // 使用者调用后将订阅回调添加进subscriptions
const removeSubscription = addSubscription(
  subscriptions,
  callback,
  options.detached,
  () => stopWatcher()
)
      // 通过watch监听store.state直接修改
const stopWatcher = scope.run(() =>
 watch(
   () => pinia.state.value[$id] as UnwrapRef<S>,
   (state) => {
     // flush默认为'pre',而在调用$patch时isListening会被设置为false,所以不会触发$patch修改state的监听回调
     // 但对于store.state直接修改的情况，store在创建完成后isSyncListening和isListening都会变成true，所以能够监听
     if (options.flush === 'sync' ? isSyncListening : isListening) {
       callback(
         {
           storeId: $id,
           type: MutationType.direct, // type为direct直接修改
           events: debuggerEvents as DebuggerEvent,
         },
         state
       )
     }
   },
   assign({}, $subscribeOptions, options)
 )
)!

return removeSubscription
 },
 $dispose,
} as _StoreWithState<Id, S, G, A>
// ...

// 整个store创建结束后将两个监听标识为true,意味着store创建完成,可以进行监听订阅操作
isListening = true
isSyncListening = true
return store
}
```

-   ### $patch修改
    

$patch用于批量修改state，内部主要通过两个标识位的修改，不触发[watch监听](https://so.csdn.net/so/search?q=watch%E7%9B%91%E5%90%AC&spm=1001.2101.3001.7020)回调

```TypeScript
function createSetupStore<
  Id extends string,
  SS extends Record<any, unknown>,
  S extends StateTree,
  G extends Record<string, _Method>,
  A extends _ActionsTree
>(
  $id: Id,
  setup: () => SS,
  options:
    | DefineSetupStoreOptions<Id, S, G, A>
    | DefineStoreOptions<Id, S, G, A> = {},
  pinia: Pinia,
  hot?: boolean,
  isOptionsStore?: boolean
): Store<Id, S, G, A> {
// internal state
let isListening: boolean // set to true at the end
let isSyncListening: boolean // set to true at the end

// ...
function $patch(
 partialStateOrMutator:
   | _DeepPartial<UnwrapRef<S>>
   | ((state: UnwrapRef<S>) => void)
): void {
 // 订阅$patch操作type
 let subscriptionMutation: SubscriptionCallbackMutation<S>
 // 避免批量修改触发$subscribe
 isListening = isSyncListening = false
 
 if (__DEV__) {
   debuggerEvents = []
 }
 // 兼容$patch传递函数、对象调用的两种调用方式
 if (typeof partialStateOrMutator === 'function') {
   partialStateOrMutator(pinia.state.value[$id] as UnwrapRef<S>)
   subscriptionMutation = {
     type: MutationType.patchFunction,  // type类型
     storeId: $id,
     events: debuggerEvents as DebuggerEvent[],
   }
 } else {
   // $patch传递对象走合并流程
   mergeReactiveObjects(pinia.state.value[$id], partialStateOrMutator)
   subscriptionMutation = {
     type: MutationType.patchObject, // type类型
     payload: partialStateOrMutator,
     storeId: $id,
     events: debuggerEvents as DebuggerEvent[],
   }
 }
 const myListenerId = (activeListener = Symbol())
 // 对于异步修改情况，异步还原isListening,让$subscribe不会监听通过$patch修改state
 nextTick().then(() => {
   if (activeListener === myListenerId) {
     isListening = true
   }
 })
 isSyncListening = true
 
 // 手动触发订阅,实现通过$patch批量修改state只触发一次订阅回调
 triggerSubscriptions(
   subscriptions,
   subscriptionMutation,
   pinia.state.value[$id] as UnwrapRef<S>
 )
}

// ...

// 整个store创建结束后将两个监听标识为true,意味着store创建完成,可以进行监听订阅操作
isListening = true
isSyncListening = true
return store
}
```

## store API

-   ## $reset
    

对于options store提供的还原初始状态的API

```TypeScript
// options store
store.$reset = function $reset() {
  const newState = state ? state() : {}  // state为options中的state
  // 通过$patch批量修改
  this.$patch(($state) => {
    assign($state, newState)
  })
}

// setup store
const $reset = __DEV__ // 开发环境下会报错
  ? () => {
      throw new Error(
        `🍍: Store "${$id}" is built using the setup syntax and does not implement $reset().`
      )
    }
  : noop
```

-   ## $dispose
    

卸载store的API

```TypeScript
// 卸载store
function $dispose() {
  scope.stop() //卸载store的scope effect
  subscriptions = []
  actionSubscriptions = []
  pinia._s.delete($id) // 从pinia上删除掉对应的store
}
```

## storeToRefs解构store

通过storeToRefs使得解构store也不会丢失响应式

-   基本使用

```TypeScript
export default defineComponent({
  setup() {
    const store = useCounterStore()
    // ❌ 这将无法生效，因为它破坏了响应性
    // 这与从 `props` 中解构是一样的。
    const { name, doubleCount } = store

    name // "eduardo"
    doubleCount // 2

    return {
      // 始终是 "eduardo"
      name,
      // 始终是 2
      doubleCount,
      // 这个将是响应式的
      doubleValue: computed(() => store.doubleCount),
      }
  },
})

// 通过storeToRefs调用
import { storeToRefs } from 'pinia'

export default defineComponent({
  setup() {
    const store = useCounterStore()
    // `name` and `doubleCount` 都是响应式 refs
    // 这也将为由插件添加的属性创建 refs
    // 同时会跳过任何 action 或非响应式(非 ref/响应式)属性
    const { name, doubleCount } = storeToRefs(store)
    // 名为 increment 的 action 可以直接提取
    const { increment } = store

    return {
      name,
      doubleCount,
      increment,
    }
  },
})
```

-   storeToRefs
    -   类似toRefs，但会跳过方法和非响应式属性

```TypeScript
// storeToRefs.ts
export function storeToRefs<SS extends StoreGeneric>(
  store: SS
): ToRefs<
  StoreState<SS> & StoreGetters<SS> & PiniaCustomStateProperties<StoreState<SS>>
> {
  if (isVue2) {
    // @ts-expect-error: toRefs include methods and others
    return toRefs(store) // vue2 版本直接all in ref
  } else {
  
    store = toRaw(store) // 拿到store原始对象

    const refs = {} as ToRefs<
      StoreState<SS> &
        StoreGetters<SS> &
        PiniaCustomStateProperties<StoreState<SS>>
    >
    for (const key in store) {
      const value = store[key]
      // 只转换ref、reactive属性
      if (isRef(value) || isReactive(value)) {
        // @ts-expect-error: the key is state or getter
        refs[key] =
          // ---
          toRef(store, key)
      }
    }

    return refs
  }
}
```

## map系列辅助函数

-   ## mapState
    

基本使用

```TypeScript
export default {
  computed: {
    // 可以访问组件中的 this.count
    // 与从 store.count 中读取的数据相同
    ...mapState(useCounterStore, ['count'])
    // 与上述相同，但将其注册为 this.myOwnName
    ...mapState(useCounterStore, {
      myOwnName: 'count',
      // 你也可以写一个函数来获得对 store 的访问权
      double: store => store.count * 2,
      // 它可以访问 `this`，但它没有标注类型...
      magicValue(store) {
        return store.someGetter + this.count + this.double
      },
    }),
  },
}
```

mapState

```TypeScript
// mapHelpers.ts
export function mapState<
  Id extends string,
  S extends StateTree,
  G extends _GettersTree<S>,
  A
>(
  useStore: StoreDefinition<Id, S, G, A>,
  keysOrMapper: any
): _MapStateReturn<S, G> | _MapStateObjectReturn<Id, S, G, A> {
  // 返回一个对象使得能够放到组件的computed属性上
  return Array.isArray(keysOrMapper)
    ? keysOrMapper.reduce((reduced, key) => {
        reduced[key] = function (this: ComponentPublicInstance) {
          // 通过useStore，因为store已经创建，所以内部会直接通过pinia._s.get(id)直接返回store而不会重新创建
          return useStore(this.$pinia)[key]
        } as () => any
        return reduced
      }, {} as _MapStateReturn<S, G>)
    : Object.keys(keysOrMapper).reduce((reduced, key: string) => {
        // @ts-expect-error
        reduced[key] = function (this: ComponentPublicInstance) {
          const store = useStore(this.$pinia)
          const storeKey = keysOrMapper[key]
          // for some reason TS is unable to infer the type of storeKey to be a
          // function
          return typeof storeKey === 'function'
            ? (storeKey as (store: Store<Id, S, G, A>) => any).call(this, store)
            : store[storeKey]
        }
        return reduced
      }, {} as _MapStateObjectReturn<Id, S, G, A>)
}
```

-   ## mapGetters
    

```TypeScript
export const mapGetters = mapState //👍
```

-   ## mapActions
    

```TypeScript
export function mapActions<
  Id extends string,
  S extends StateTree,
  G extends _GettersTree<S>,
  A,
  KeyMapper extends Record<string, keyof A>
>(
  useStore: StoreDefinition<Id, S, G, A>,
  keysOrMapper: Array<keyof A> | KeyMapper
): _MapActionsReturn<A> | _MapActionsObjectReturn<A, KeyMapper> {

  return Array.isArray(keysOrMapper)
    ? keysOrMapper.reduce((reduced, key) => {
        // 和mapState类似,闭包存了下this和其他参数
        reduced[key] = function (
          this: ComponentPublicInstance,
          ...args: any[]
        ) {
          return useStore(this.$pinia)[key](...args)
        }
        return reduced
      }, {} as _MapActionsReturn<A>)
    : Object.keys(keysOrMapper).reduce((reduced, key: keyof KeyMapper) => {
        // @ts-expect-error
        reduced[key] = function (
          this: ComponentPublicInstance,
          ...args: any[]
        ) {
          return useStore(this.$pinia)[keysOrMapper[key]](...args)
        }
        return reduced
      }, {} as _MapActionsObjectReturn<A, KeyMapper>)
}
```

## 和vuex区别

-   ## 不再有嵌套结构的模块
    
    -   看过vuex的应该知道，整个vuex就是个嵌套的大对象，会根据命名空间一步一步从对象中取出内容，不过内部会帮你拼接命名空间路径，实际调用时也还好
    -   而pinia不再通过命名空间来嵌套对象，通过pinia.\_s这个Map结构来存储创建的store，通过pinia.state.value\[storeId\]来存储state，整个就是一平级的结构
-   ## 创建非常方便，和写一个hook函数一样轻松
    
    -   虽然但是，总觉得vue越来越像React，用过hox的应该知道，Pinia的使用方式几乎和hox一模一样。。。
-   ## 无需mutation
    
    -   见仁见智吧，有个mutation调用流程更规范，没有就是函数调用，无学习成本
-   总结一下，虽然单项数据流类型的状态管理库（redux、vuex）具有十分严谨的调用步骤，但这种函数式的方式才是未来（主要是开发爽了。。。）