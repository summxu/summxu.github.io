---
title: 基于 document.execCommand 的富文本
date: 2019-04-30 21:05:35
tags:
    - JavaScript
---
# 前言
最近在项目中准备集成一个富文本编辑器，原来使用的是 Quill，之后发现项目打包体积瞬增了 200kb。虽然 Quill 完全能够满足项目需求，但其提供的诸多功能是用不上的，可以使用更轻量的实现代替。

在寻找新的替代品时，也顺便瞄了下各种编辑器的内部实现。一些体积庞大的编辑器一般都兼容低版本浏览器，不得不写很多兼容性的代码。而一些 MINI、轻量的编辑器是不对低端浏览器作兼容的，使用`Selection`、`Range`或者`document.execCommand`实现。

之前在 JavaScript 获取输入时的光标位置及场景问题 中提到过`Selection`和`Range`，这次就说说`document.execCommand`。

# document.execCommand
该方法可以对可编辑器区域进行操作，比如加粗文字、改变字号、插入链接等。可编辑区域也就是设置了`contentEditable`属性的元素。

```
// 语法
bool = document.execCommand(commandName, showDefaultUI[, arg])
```
* `commandName`：String，命令的名称
* `showDefaultUI`：Boolean，是否展示用户界面（暂时没有发现“用户界面”指的是什么），一般为false
* `arg`：String，要传递的参数，比如插入链接需要传入一个 URL，可选
* 返回值：Boolean，表示操作是否支持或是否启用
假如要把选中的文字加粗，只需要这样：

`document.execCommand('bold', false)`
# 兼容性
[http://static.imys.net/execCommand-support-2017113.jpg]

对于现代浏览器的兼容度较好，不过 Android 53 取消兼容的什么鬼？

除了了解该方法的兼容性外，还需要了解每一种可执行命令的兼容性。比如`decreaseFontSize`，给选中文字添加`small`标签，也就是小号字体，谷歌是不支持的。

可以使用以下方法检测命令是否在浏览器中支持：

`document.queryCommandSupported(commandName)`
# 简陋的实现
不到 50 行代码，基于[document.execCommand实现富文本编辑器](https://imys.net/demo/cmdEditor.html)。

有兴趣的同学可以以此为出发点，去开发一个具备完整功能的编辑器。懒得折腾的可以像我一样多搜索几下，找一个合适的轮子。😅

参考
[MDN: document.execCommand](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/execCommand)