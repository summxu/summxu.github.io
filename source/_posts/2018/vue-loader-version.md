---
layout:     post
title:      "Vue-loader版本遇到的坑"
subtitle:   "Vue-loader@15"
date:       2018-07-22
author:     "BoomXu"
tags:
    - Vue
    - WebPack
---

> 在学习Vue.js和Webpack的路上遇到了不少问题,也找到了相对的解决办法, 这次记录下在vue-loader版本遇到的问题.

# vue-loader简介
在webpack中加载第三方的文件支持,就要有第三方的文件解析工具,在vue的组件模板文件中,要有vue-loader的支持才可以.他可以将vue文件转换为JS模块；
# vue-loader特性
（1）ES2015默认支持 
（2）允许对VUE组件的组成部分使用其他webpack loader;比如对< style >使用SASS（编译CSS语言），对< template >使用JADE（jade是一个高性能的模板引擎，用JS实现，也有其他语言的实现—php,scala,yuby,python,java，可以供给node使用） 
（3）.vue文件中允许自定义节点，然后使用自定义的loader处理他们 
（4）对< style >< template >中的静态资源当做模块来对待，并且使用webpack loaders进行处理 
（5）对每个组件模拟出CSS作用域 
（6）支持开发期组件的热重载 
在编写vue应用程序时，组合使用webpack跟vue-loader能带来一个现代。灵活并且非常强大的前端工作流程；
# vue-loader@14 Version
特性上面说了,**允许对VUE组件的组成部分使用其他webpack loader;比如对< style >使用SASS（编译CSS语言），对< template >使用JADE**，但是我在运行的过程中在vue组件模板文件下写stylus的样式一直报错
![](http://p5oqx8gut.bkt.clouddn.com/18-7-22/21405627.jpg)
![](http://p5oqx8gut.bkt.clouddn.com/18-7-22/28336505.jpg)
总是提示说需要装一个loader支持，但是我已经装了
![](http://p5oqx8gut.bkt.clouddn.com/18-7-22/55563364.jpg)
而且语法也有写错
![](http://p5oqx8gut.bkt.clouddn.com/18-7-22/80861378.jpg)
这是我看到报错里提到了vue-loader，果然是vue-loader 的版本问题
# vue-loader@15 Version
在最新的npm上装的是vue-loader 15.x 版本，但是现在直接npm装的最新版本的vue-loader 15 把这个功能给去除了,同时在vue-loader 15配置的时候还要引入一个插件
```
15版本以上的vue-loader 需要插件支持 , 插件是个对象,这里引入的时候要转换成对象才行
const {VueLoaderPlugin} = require('vue-loader')


plugins: [
  //15版本以上的vue-loader 需要插件支持
  new VueLoaderPlugin()
],
```

不仅如此，15版本的vue-loader还 **去除了 对< style > 编译CSS预处理的支持**

现在暂时使用14版本的vue-loader，15版本的新特性和使用方法还需要进一步的学习。。。