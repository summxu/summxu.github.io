---
title: Vue3响应式属性Reactive和Ref
date: 2023-03-27 08:49:38
tags:
  - Vue
---

## ref 的基本使用

1.ref:接受一个内部值并返回一个响应式且可变的 ref 对象。ref 对象仅有一个 `.value` property，指向该内部值。

**注意**：被ref包装之后需要.value 来进行赋值，因为使用ref包装，返回的是一个对象，Ref TS对应的接口

```TypeScript
<template>
  <button @click="onChangeMsg">修改数据</button>
  <div>{{ message }}</div>
</template>

<script setup lang="ts">
import { ref, Ref } from 'vue';
const message: Ref<string> = ref('ref响应式数据');

const onChangeMsg = () => {
  message.value = '修改后的数据';
};
</script>
```

ts的另外一种方式：

```TypeScript
<template>
  <button @click="onChangeMsg">修改数据</button>
  <div>{{ message }}</div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
const message = ref<string | number>('ref响应式数据');

const onChangeMsg = () => {
  message.value = '修改后的数据';
};
</script>
```

3.shallowRef创建一个跟踪自身 `.value` 变化的 ref，但不会使其值也变成响应式的。<template\>


```TypeScript
<template>
  <button @click="onChangeMsg">修改数据</button>
  <div>{{ message }}</div>
</template>

<script setup lang="ts">
import { ref, shallowRef } from 'vue';
type Obj = {
  name: string;
  num: number;
};
const message = shallowRef<Obj>({
  name: 'vue3',
  num: 100
});

const onChangeMsg = () => {
  //修改的值无法显示在页面上
  message.value.name = '修改了name';
  //修改的值可以显示在页面上
  //message.value = { name: '修改了name', num: 123 };　
  //triggerRef强制修改，修改的值可以在页面显示  
  triggerRef(message);
  console.log(message, 'message');
};
</script>
```

4.customRef 是个工厂函数要求我们返回一个对象 并且实现 get 和 set：

应用场景：实现防抖函数
```TypeScript
<template>
  <button @click="onChangeMsg">修改数据</button>
  <div>{{ message }}</div>
</template>
 
<script setup lang="ts">
import { ref, shallowRef, triggerRef, customRef } from 'vue';
 
let message = MycustomRef('我是原始数据');
const onChangeMsg = () => {
  message.value = '修改了数据';
};
 
function MycustomRef<T>(value: T) {
  return customRef((track, trigger) => {
    return {
      get: () => {
        track();//通知vue，跟踪数据的变化
        return value;
      },
      set: (newVal: T) => {
        value = newVal;
        trigger();//通知vue重新解析模版，挂载数据
      }
    };
  });
}
</script>
```

## ref 为何要用.value

> 在Vue2中，所有的数据都通过一个Data进行统一的返回，并且在data中对某个组件要用的数据进行统一的管理，常见的使用形式是这样的：

```TypeScript
<template>
  <div class="div">
    <todos :Obj="tos" :removeObj="removeObj"></todos>
  </div>
</template>
<script>
import search from '@/components/search'
import todos from '@/components/todos'
import all from '@/components/all'
export default {
  name: 'App',
  data () {
    return {
      tos: [
        { id: '001', value: '第一个', done: true },
        { id: '002', value: '第二个', done: true },
        { id: '003', value: '第三个', done: false },
        { id: '004', value: '第四个', done: true },
      ],
    }
  },
  computed: { },
  components: {
    search, todos, all,
  },
  methods: {
    removeObj (obj) {
      console.log(obj.id)
      this.tos = this.tos.filter(item => item.id !== obj.id)
      console.log(this.tos)
    },
  },
}
</script>
```

可以看出来这里定义的内容都在一个数组中进行，或者是一个函数，将要使用的数据返回出来，这里无论怎么进行操作处理，最终进行数据代理的时候得到的都是一个对象，Vue2中直接通过defineProperty进行处理，并绑定对应的监听事件进行响应式的处理。

而Vue3中，数据的定义可以是单独的，Vue可以让随时需要随时定义，这也就带来了另一个问题，我需要的一个数据可能不是对象

**如果要定义的数据不是对象，还需要代理会怎么样？**

-   `Object.defineProperty()` 方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性，并返回此对象。

通过这个可以明确看出，只能进行对象的代理，不能进行普通数据的代理

在Vue3中数据代理可以使用单一数据了，并且也改进了数据代理的方式，使用的是`Proxy`完成了数据代理，而MDN中对`Proxy`也进行了定义：

-   `Proxy` 对象用于创建一个对象的代理，从而实现基本操作的拦截和自定义（如属性查找、赋值、枚举、函数调用等）。

即使是Vue3中使用的`Proxy`的代理方式也不能进行普通数据的代理，所以当调用Ref的时候其实仍然创建了一个`Proxy`对象，并且Vue帮你给这个对象了一个value属性，属性值就是你定义的内容，改变的时候监视的改变依然是通过`Proxy`的数据劫持来进行响应式的处理，而模板中使用的时候Vue会默认调用对应的value属性，从而完成模板中的内容的直接调用 


## Reactive 的基本使用

1.reactive:用来绑定复杂的数据类型：数组，对象等。

**注意**:reactive如果绑定的是基础类型数据会报错。

```TypeScript
<template>
  <button @click="onChangeMsg">修改数据</button>
  <div>{{ obj.name }}</div>
</template>

<script setup lang="ts">
import { reactive } from 'vue';
let obj = reactive({
  name: '张三',
  age: 18,
  boj: {
    namespaced: true
  }
});
setTimeout(() => {
  obj.name = '异步赋值无效';
}, 2000);
const onChangeMsg = () => {
  obj.name = '李四';
};
</script>
```

2.readonly:拷贝一份proxy对象并设置为只读属性

```TypeScript
<template>
  <button @click="onChangeMsg">修改数据</button>
  <div>{{ obj.name }}</div>
</template>

<script setup lang="ts">
import { reactive, readonly } from 'vue';
let obj = reactive({
  name: '张三',
  age: 18,
  boj: {
    namespaced: true
  }
});
let CopyObj = readonly(obj);
setTimeout(() => {
  CopyObj.name = '异步赋值无效';
}, 2000);
const onChangeMsg = () => {
  CopyObj.name = '李四';
};
</script>
```

3.shallowReactive:拷贝一份对象，可以修改浅层数据，无法修改深层数据

```TypeScript
<template>
  <button @click="onChangeMsg">修改数据</button>
  <div>{{ obj.name }}</div>
</template>

<script setup lang="ts">
import { reactive, shallowReactive } from 'vue';
let obj = reactive({
  name: '张三',
  age: 18,
  boj: {
    namespaced: true
  }
});
let CopyObj = shallowReactive(obj);
setTimeout(() => {
  //可以修改
  CopyObj.name = '异步赋值';
  //无法修改
  CopyObj.boj.namespaced = false;
}, 2000);
console.log(CopyObj.boj.namespaced);
const onChangeMsg = () => {
  CopyObj.name = '李四';
};
</script>
```