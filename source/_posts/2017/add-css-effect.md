---
layout:     post
title:      "通过CSS添加网页的动画效果"
subtitle:   "用CSS美化了下博客文章页的标题个tags,主要运用到CSS效果."
date:       2017-06-16 14:03:56
author:     "BoomXu"
header-img: "https://ooo.0o0.ooo/2017/06/16/59439d8b17cf3.jpg"
tags:
    - CSS
    - 博客
---



> 昨天写了[通过jQuery添加网页的动画效果](http://www.BoomXu.com/2017/06/15/add-jquery-effect/)来美化了一下博客，发现jQuery的动画效果**卡出翔**，别说美观了，用户体验都没保证，今天更新下**用CSS添加网页特效**，顺便记录下自己的CSS的学习；。

在网上查阅相关CSS动画特效的资料，发现了一个CSS样式库：**Aninmate.css**
这次主要是对这个样式库进行介绍，里面有很多好看的动画样式。

# Aninmate.css

这里有个Aninmate样式的各个效果预览：**[点我查看](https://yuyu147.github.io/CSS/aninmatedemo.html)**

我打包了Aninmate.css的引入文件几Demo：**[在此下载](http://or8fqs2b1.bkt.clouddn.com/animate.rar)**

## 简介

**animate.css** 是一个来自国外的 CSS3 动画库，它预设了抖动（shake）、闪烁（flash）、弹跳（bounce）、翻转（flip）、旋转（rotateIn/rotateOut）、淡入淡出（fadeIn/fadeOut）等多达 **60 多种动画效果** ，几乎包含了所有常见的动画效果。

虽然借助 animate.css 能够很方便、快速的制作 CSS3 动画效果，但还是 **建议看看** animate.css 的代码，也许你能从中学到一些东西。

## 兼容

浏览器兼容：当然是只兼容支持 CSS3 animate 属性的浏览器，他们分别是：**IE10+、Firefox、Chrome、Opera、Safari**。

## 使用方法

1. 当然首先要引入样式文件

```
<link rel="stylesheet" href="animate.min.css">
```

2. HTML 及使用

```
<div class="animated bounce" id="dowebok"></div>
```

给元素加上 class 后，刷新页面，就能看到动画效果了。animated 类似于全局变量，它定义了动画的持续时间；bounce 是动画具体的动画效果的名称，你可以选择任意的效果。

如果动画是无限播放的，可以添加 class infinite。

你也可以通过 JavaScript 或 jQuery 给元素添加这些 class，比如：

```
$(function(){
    $('#dowebok').addClass('animated bounce');
});
```

有些动画效果最后会让元素不可见，比如淡出、向左滑动等等，可能你又需要将 class 删除，比如：

```
$(function(){
    $('#dowebok').addClass('animated bounce');
    setTimeout(function(){
        $('#dowebok').removeClass('bounce');
    }, 1000);
});
```

animate.css 的默认设置也许有些时候并不是我们想要的，所以你可以重新设置，比如：

```
#dowebok {
    animate-duration: 2s;    //动画持续时间
    animate-delay: 1s;    //动画延迟时间
    animate-iteration-count: 2;    //动画执行次数
}
```

3. Github地址

<https://github.com/daneden/animate.css>

## 我的引用代码

本次引用了两个页面，一个文章Post页，一个是博客首页：

**post页：**

```
<a class="tag animated bounceInDown" href="{ { site.baseurl } }/tags/#{ { tag } }" title="{ { tag } }">{ { tag } }</a>
<h1 class="animated rotateInDownLeft">{ { page.title } }</h1>
<h2 class="subheading animated rotateInUpRight">{ { page.subtitle } }</h2>
```

**首页：**

```
<h1 class="animated flipInX">{% if page.title %}{ { page.title } }{% else %}{ { site.title } }{% endif %}</h1>
<span class="subheading animated flipInY">{ { page.description } }</span>
```

# 为什么jQuery会卡顿?

让我们从基本开始说起： Javascript 和 jQuery 两者不能混为一谈。Javascript 动画很快，而 jQuery 动画很慢。为什么呢？因为尽管 jQuery 异常强大，但是它的设计目标并不是一个高效的动画引擎：

* jQuery 不能避免 layout thrashing （有人喜欢将其翻译为“布局颠簸”，会导致多余relayout/reflow），因为它的代码不仅仅用于动画，它还用于很多其他场景。
* jQuery的内存消耗较大，经常会触发垃圾回收。而垃圾回收触发时很容易让动画卡住。
* jQuery使用了setInterval而不是 reqeustAnimationFrame(RAF)，因为 RAF 会在窗口失去焦点时停止触发，这会导致jQuery的bug。（目前jQuery已经使用了RAF）

注意 layout thrashing 会导致动画在开始的时候卡顿，垃圾回收的触发会导致动画运行过程中的卡顿，不使用 RAF 则会导致动画帧率低。

# 总结

CSS3也是个很强大的东西，前端的资源真的是很多很多。没想到还有专门的动画效果CSS库，在以后还会更加灵活的去运用其他第三方的库。