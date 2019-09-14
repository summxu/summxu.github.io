---
layout:     post
title:      "Vue组件间样式污染大坑"
subtitle:   "scoped"
date:       2018-12-04
author:     "BoomXu"
tags:
    - Vue
---

我们都知道，Vue 组件内的样式可以写在 Style 标签下，而各组件之间的样式冲突(污染)问题也十分常见，当然我们可以尽量避免起相同的类名，但项目较大的时候，Class类名时而会冲突。

当然 Vue官方也给了我们解决方法，就是定义了 scoped 这个属性的设置：

> 这个可选 scoped 属性会自动添加一个唯一的属性 (比如 data-v-21e5b78) 为组件内 CSS 指定作用域，编译的时候 .list-container:hover 会被编译成类似 .list-container[data-v-21e5b78]:hover。

> 最后，Vue 的单文件组件里的样式设置是非常灵活的。通过 vue-loader，你可以使用任意预处理器、后处理器，甚至深度集成 CSS Modules——全部都在 <style> 标签内。

但是问题就在于 **虽然加了scoped,但是却仍热无法锁住用@import引入的外部css文件**

这里的解决方法也比较神奇，就是把引入的css文件改为使用预处理器处理的 less sass 或者 styl 文件，具体原理暂时还不得而知

使用 /deep/ 来解决更深一级的标签