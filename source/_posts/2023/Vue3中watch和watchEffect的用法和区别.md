---
title: Vue3中watch和watchEffect的用法和区别
date: 2023-01-12 17:13:34
tags:
  - Vue
---

### 1.1 watch 基本使用

在 Vue3 中的组合式 API 中，watch 的作用和 Vue2 中的 watch 作用是一样的，他们都是用来监听响应式状态发生变化的，当响应式状态发生变化时，都会触发一个回调函数。

**代码如下：**

```TypeScript
<template>
  <img alt="Vue logo" src="./assets/logo.png" />
  <p>{{ message }}</p>
  <button @click="changeMsg">更改 message</button>
</template>
<script setup lang="ts">
import { ref, watch } from "vue";


const message = ref("李四");
watch(message, (newValue, oldValue) => {
  console.log("新的值:", newValue);
  console.log("旧的值:", oldValue);
});
const changeMsg = () => {
  message.value = "张三";
};
</script>
```

上段代码中点击按钮就会更改响应式变量 message 的值。又使用 watch 监听器监听了 message 变量，当它发生变化时，就会触发 watch 监听函数中的回调函数，并且回调函数默认接收两个参数：新值和旧值。

**注意：当第一进入页面时，watch 监听函数的回调函数是不会执行的。**

### 1.2 watch 监听类型

前面一直强调 watch 监听的是响应式数据，如果监听的数据不是响应式的，那么可能会抛出警告。

**（1）ref 和计算属性**

ref 定义的数据是可以监听到的，因为前面的代码以及证明了。除此之外，计算属性也是可以监听到的，比如下列代码：

```TypeScript
const message = ref("李四");
const newMessage = computed(() => {
  return message.value;
});
watch(newMessage, (newValue, oldValue) => {
  console.log("新的值:", newValue);
  console.log("旧的值:", oldValue);
});
```

当 message 发生变化时，计算属性 newMessage 也会重新计算得出新的结果， watch 监听函数是可以监听到计算属性变化的。

**（2）getter 函数**

这里的 getter 函数可以简单的理解为获取数据的一个函数，说白了该函数就是一个返回值的操作，有点类似与计算属性。

**示例代码如下：**

```TypeScript
<template>
  <img alt="Vue logo" src="./assets/logo.png" />
  <p>{{ message }}</p>
  <p>{{ x1 + x2 }}</p>
  <button @click="changeMsg">更改 message</button>
</template>
<script setup lang="ts">
import { ref, watch } from "vue";


const message = ref("李四");
const x1 = ref(12);
const x2 = ref(13);
watch(
  () => x1.value + x2.value,
  (newValue, oldValue) => {
    console.log("新的值:", newValue);
    console.log("旧的值:", oldValue);
  }
);
const changeMsg = () => {
  message.value = "张三";
  x1.value = 14;
  x2.value = 23;
};
</script>
```

上段代码中 watch 监听器中的第一个参数是一个箭头函数，也就是 getter 函数，getter 函数返回的是响应式数据 x1 和 x2 相加的值，当这两个中中有一个变化，都会执行 watch 中的回调函数。有点像是直接把计算属性写到监听器里面去了。

**（3）监听响应式对象**

前面监听的都是值类型的响应式数据，同样也可以监听响应式的对象。

**代码如下：**

```TypeScript
const number = reactive({ count: 0 });
const countAdd = () => {
  number.count++;
};
watch(number, (newValue, oldValue) => {
  console.log("新的值:", newValue);
  console.log("旧的值:", oldValue);
});
```

当 watch 监听的是一个响应式对象时，会隐式地创建一个深层侦听器，即该响应式对象里面的任何属性发生变化，都会触发监听函数中的回调函数。

需要注意的，watch 不能直接监听响应式对象的属性，即下面的写法是错误的：

```TypeScript
const number = reactive({ count: 0 });
const countAdd = () => {
  number.count++;
};
watch(number.count, (newValue, oldValue) => {
  console.log("新的值:", newValue);
  console.log("旧的值:", oldValue);
});
```

上段代码中相当于你直接向 watch 传递了一个非响应式的数字，然而 watch 只能监听响应式数据。

**但是：**

如果非要监听响应式对象中的某个属性，可以使用 getter 函数的形式，代码如下：

```TypeScript
watch(
  () => number.count,
  (newValue, oldValue) => {
    console.log("新的值:", newValue);
    console.log("旧的值:", oldValue);
  }
);
```

上段代码也是可以监听到 count 变化的。

**（4）监听多个来源的数组**

watch 还可以监听数组，前提是这个数组内部含有响应式数据。

**代码如下：**

```TypeScript
const x1 = ref(12);
const number = reactive({ count: 0 });
const countAdd = () => {
  number.count++;
};
watch([x1, () => number.count], (newValue, oldValue) => {
  console.log("新的值:", newValue);
  console.log("旧的值:", oldValue);
});
```

### 1.3 深度监听

在前面的代码中，如果将一个响应式对象传递给 watch 监听器时，只要对象里面的某个属性发生了变化，那么就会执行监听器回调函数。

究其原因，因为传入响应对象给 watch 时，隐式的添加一个深度监听器，这就让造成了牵一发而至全身的效果。

但是，如果是使用的 getter 函数返回响应式对象的形式，那么响应式对象的属性值发生变化，是不会触发 watch 的回调函数的。

**代码如下：**

```TypeScript
const number = reactive({ count: 0 });
const countAdd = () => {
  number.count++;
};
watch(
  () => number,
  (newValue, oldValue) => {
    console.log("新的值:", newValue);
    console.log("旧的值:", oldValue);
  },
);
```

上段代码中使用 getter 函数返回了响应式对象，当更改 number 中 count 的值时，watch 的回调函数是不会执行的。

为了实现上述代码的监听，可以手动给监听器加上深度监听的效果。

**代码如下：**

```TypeScript
const number = reactive({ count: 0 });
const countAdd = () => {
  number.count++;
};
watch(
  () => number,
  (newValue, oldValue) => {
    console.log("新的值:", newValue);
    console.log("旧的值:", oldValue);
  },
  { deep: true }
);
```

添加深度监听很简单，只需要给 watch 添加第三个参数即可：{ deep: true }。

**注意：**上段代码中的 newValue 和 oldValue 的值是一样的，除非把响应式对象即 number 整个替换掉，那么这两个值才会变得不一样。除此之外，深度监听会遍历响应式对象的所有属性，开销较大，当对象体很大时，需要慎用**。**

**所以推荐 getter 函数只返回相应是对象中的某一个属性！！**

## 2.watchEffect

前面使用 watch 监听数据状态时，不知道有没有发现这样一个问题：只有当监听的数据源发生了变化，监听函数的回调函数才会执行。但是需求总是多变的，有些场景下可能需要刚进页面，或者说第一次渲染页面的时候，watch 监听器里面的回调函数就执行一遍。

面对这种需求怎样处理呢？一般有两种方式：

**方式一：**

这种方式也是通过 watch 实现的，确切的说是巧妙的实现，而不是依赖于 watch 监听器。

**代码如下：**

```TypeScript
const number = reactive({ count: 0 });
// 进入页面先执行一遍
const callback = () => {
  console.log("新的值:", number.count);
  console.log("旧的值:", number.count);
};
callback();
watch(
  () => number.count,
  (newValue, oldValue) => {
    console.log("新的值:", newValue);
    console.log("旧的值:", oldValue);
  },
  { deep: true }
);
```

既然想要第一次进入页面的时候就执行一遍回调函数，那么不妨把回调函数直接提取出来，进入页面执行一遍即可，这也算是巧妙的实现了的需求。

但是这种方式似乎不太优雅，而且有些繁琐。所以 Vue 推出了更加优雅的方法：watchEffect 监听器。

**方式二：**

watchEffect 也是一个监听器，只不过它不会像 watch 那样接收一个明确的数据源，它只接收一个回调函数。而在这个回调函数当中，它会自动监听响应数据，当回调函数里面的响应数据发生变化，回调函数就会立即执行。

所以可以将方式一中的代码使用 watchEffect 优雅的实现。

**代码如下：**

```TypeScript
const number = reactive({ count: 0 });
const countAdd = () => {
  number.count++;
};
watchEffect(()=>{
  console.log("新的值:", number.count);
})
```

上段代码中，当第一次进入页面时，number 响应数据从无到有，这个时候就会触发 watchEffect 的回调函数，因为在 watchEffect 回调函数中使用了 number 响应数据，所以它会自动跟踪 number 数据的变化。当点击按钮更改 count 的值时，watchEffect 中的回调函数便会再次执行。

这样代码是不是简单很多呀！

## 3.watch 和 watchEffect 区别

已经大概知道了 watch 和 watchEffect 的用法，那么它们之间的区别相信也了解了一些，这里总结一下它们之间的区别。

-   watch 和 watchEffect 都能监听响应式数据的变化，不同的是它们监听数据变化的方式不同。
-   watch 会明确监听某一个响应数据，而 watchEffect 则是隐式的监听回调函数中响应数据。
-   watch 在响应数据初始化时是不会执行回调函数的，watchEffect 在响应数据初始化时就会立即执行回调函数。

## 4.回调中的 DOM

如果在监听器的回调函数中或取 DOM，这个时候的 DOM 是更新前的还是更新后的？

不妨实验一下。

**代码如下：**

```TypeScript
<template>
  <img alt="Vue logo" src="./assets/logo.png" />
  <p ref="msgRef">{{ message }}</p>
  <button @click="changeMsg">更改 message</button>
</template>
<script setup lang="ts">
import { computed, reactive, ref, watch, watchEffect } from "vue";


const message = ref("李四");
const msgRef = ref<any>(null);
const changeMsg = () => {
  message.value = "张三";
};
watch(message, (newValue, oldValue) => {
  console.log("DOM 节点", msgRef.value.innerHTML);
  console.log("新的值:", newValue);
  console.log("旧的值:", oldValue);
});
</script>
```

通过点击按钮更改 message 的值，从“李四”变为“张三”。但是发现在监听器的回调函数里面获取到的 DOM 元素还是“李四”，说明 DOM 还没有更新。

**解决方法：**

如果想要在回调函数里面获取更新后的 DOM，非常简单，只需要再给监听器多传递一个参数选项即可：flush: 'post'。watch 和 watchEffect 同理。

**代码如下：**

```TypeScript
watch(source, callback, {
  flush: 'post'
})
watchEffect(callback, {
  flush: 'post'
})
```

**修改后的代码：**

```TypeScript
watch(
  message,
  (newValue, oldValue) => {
    console.log("DOM 节点", msgRef.value.innerHTML);
    console.log("新的值:", newValue);
    console.log("旧的值:", oldValue);
  },
  {
    flush: "post",
  }
);
```

这个时候在回调函数中获取到的已经是更新后的 DOM 节点了。

**补充：**

虽然 watch 和 watchEffect 都可以用上述方法解决 DOM 问题，但是 Vue3 单独给 watchEffect 提供了一个更方便的方法，也可以叫做 watchEffect 的别名，代码如下：

```TypeScript
watchPostEffect(() => {
  /* 在 Vue 更新后执行 */
})
```

## 5.手动停止监听器

通常来说，的一个组件被销毁或者卸载后，监听器也会跟着被停止，并不需要手动去关闭监听器。但是总是有一些特殊情况，即使组件卸载了，但是监听器依然存在，这个时候其实式需要手动关闭它的，否则容易造成内存泄漏。

比如下面这中写法，就需要手动停止监听器：

```TypeScript
<script setup>
import { watchEffect } from 'vue'
// 它会自动停止
watchEffect(() => {})
// ...这个则不会！
setTimeout(() => {
  watchEffect(() => {})
}, 100)
</script>
```

上段代码中采用异步的方式创建了一个监听器，这个时候监听器没有与当前组件绑定，所以即使组件销毁了，监听器依然存在。

关闭方法很简单，**代码如下：**

```TypeScript
const unwatch = watchEffect(() => {})
// ...当该侦听器不再需要时
unwatch()
```

需要用一个变量接收监听器函数的返回值，其实就是返回的一个函数，然后调用该函数，即可关闭当前监听器。
