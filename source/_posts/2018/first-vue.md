---
layout:     post
title:      "初次尝试Vue"
subtitle:   "Vue.js"
date:       2018-04-23
author:     "BoomXu"
tags:
    - Vue
---

> 我换了个Jekyll 随机自动换图的接口，以后不用每次都上传图片了 。。 https://img.xjh.me/random_img.php?type=bg&ctype=nature&return=302 速度还很快 天天好心情

# VUE2简介

## 数据绑定

省去js操作dom，动态的把js的数据绑定到html，数据一旦改变，html数据跟着改变

## vue文件开发方式

```
<template>

</template>

<script>

</script>

<style>

</style>
```

组件化开发，可以自定义标签，包含html、js、css 。。 在一个文件里面直接写各种组建。

## render方法

组件数据变化时vue会自动用render方法去遍历构建`<template>`标签内的标签节点。

# API重点

## 生命周期方法

组件什么时候编译，什么时候生html元素，都是有规定的生命周期方法。控制生命周期方法可以方便的在某个地方调用某种功能

## computed

数据改变的时候调用的方法，方便的控制数据的变化

# Vue之前装的pack

## autoprefixer

这个包是省去了写css的时候对特定的浏览器的写法，它可以自动填补上。

## Postcss

一个css插件系统,后期用插件处理的css处理器 具体可以看这个 

<https://segmentfault.com/a/1190000011595620>

## babel

用来演示 vue-jsx 的操作