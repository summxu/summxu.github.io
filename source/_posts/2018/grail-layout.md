---
layout:     post
title:      "Overflow实现圣杯布局"
subtitle:   "双飞翼"
date:       2018-08-24
author:     "BoomXu"
tags:
    - CSS
---

> 工作过程中遇到了很多问题，也学到了很多知识，这次用Overflow来触发BFC以实现双飞翼布局，很简单的一件事。但中间却有点学问。


# 实现原理

所谓原理就是用Overflow来触发BFC（块级格式化范围）。

1.这是 W3C CSS 2.1 规范中的一个概念，它决定了元素如何对其内容进行定位，以及与其他元素的关系和相互作用。当涉及到可视化布局的时候，Block Formatting Context提供了一个环境，HTML元素在这个环境中按照一定规则进行布局。一个环境中的元素不会影响到其它环境中的布局。比如浮动元素会形成BFC，浮动元素内部子元素的主要受该浮动元素影响，两个浮动元素之间是互不影响的。这里有点类似一个BFC就是一个独立的行政单位的意思。也可以说BFC就是一个作用范围。可以把它理解成是一个独立的容器，并且这个容器的里box的布局，与这个容器外的毫不相干。

2.另一个通俗点的解释是：在普通流中的 Box(框) 属于一种 formatting context(格式化上下文) ，类型可以是 block ，或者是 inline ，但不能同时属于这两者。并且， Block boxes(块框) 在 block formatting context(块格式化上下文) 里格式化， Inline boxes(块内框) 则在 inline formatting context(行内格式化上下文) 里格式化。任何被渲染的元素都属于一个 box ，并且不是 block ，就是 inline 。即使是未被任何元素包裹的文本，根据不同的情况，也会属于匿名的 block boxes 或者 inline boxes。所以上面的描述，即是把所有的元素划分到对应的 formatting context 里。

总的来说BFC是和浮动划分开来，两者互不干扰，所以在center 中间的元素 宽度 100 % 不会铺满整个窗口

# 实现代码


``` HTML
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>overflow</title>
  <style>
    .left {
      float: left;
      width: 100px;
      height: 100px;
      background-color: skyblue;
    }
    .right {
      float: right;
      width: 100px;
      height: 100px;
      background-color: skyblue;
    }
    .center {
      overflow: hidden;
      height: 100px;
      background-color: pink;
      line-height: 100px;
    }
    .search {
      height: 50%;
      width: 100%;
    }
  </style>
</head>
<body>
  <div class="left"></div>
  <div class="right"></div>
  <div class="center">
    <input class="search" type="text">
  </div>
</body>
</html>
```
但是要注意，left center 和 right 标签的位置顺序不能放错。