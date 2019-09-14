---
layout:     post
title:      "初次尝试Vue Jsx"
subtitle:   "Vue.js"
date:       2018-04-24
author:     "BoomXu"
tags:
    - Vue
---

## Vue声明加载

```
<script>
import Header from './todo/header.vue'
import Footer from './todo/footer.jsx'

export default {
  components:{
    Header,
    Footer,
  }
}
</script>
```

这里用 components 方法来声明加载过来的外部文件。。

## Jsx


> React 使用 JSX 来替代常规的 JavaScript。
> JSX 是一个看起来很像 XML 的 JavaScript 语法扩展。
> 我们不需要一定使用 JSX，但它有以下优点：
> JSX 执行更快，因为它在编译为 JavaScript 代码后进行了优化。
它是类型安全的，在编译过程中就能发现错误。
使用 JSX 编写模板更加简单快速。

jsx更好的把js和html结合起来，可以很简单的做到原生js很复杂的操作，下面是一段代码：

``` JavaScript
import '../assets/styles/footer.styl'

export default{
  data() {
    return {
      author: 'BoomXu'
    }
  },
  render() {
    return(
      <div id="footer">
        <span>Written by {this.author}</span>    //这里用了this对像调用
      </div>
    )
  }
}
```

jsx在生成html的时候用了 ` render()`方法，其实vue和这个一样，vue里的 `<template>`标签其实就是用了` render()`方法来生成的html结构。

但是jsx不好的一点在于他不能直接写 **style** 需要引入外部文件才可以。

`methods{}` 声明方法