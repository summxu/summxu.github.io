---
layout:     post
title:      "巧用VScode“用户代码片段”来提高效率"
subtitle:   "自定义VS code来实现bootstrap和jQuery的一键引入！"
date:       2017-06-10
author:     "BoomXu"
header-img: "https://ooo.0o0.ooo/2017/06/10/593bb2bd64179.jpg"
tags:
    - 技术分享
    - HTML
    - BootStrap
---



# 前言

刚学前端，用VS code写代码很爽，而且还支持**git**、**markdown**可以直接上传刚写完的博客。而用来写HTML、JavaScript也是相当方便。最近在学习bootstrap，总感觉每次引入js或者样式文件非常麻烦，但发现VS code有**代码片段**功能，在此记录下。

# 使用

## 找到入口

这里提供了两种方法：

1. 按「Alt」键切换菜单栏，通过文件 > 首选项 > 用户代码片段，选择进入目的语言的代码段设置文件；

    ![123](https://ooo.0o0.ooo/2017/06/10/593bbc44c73ff.png)
2. 通过快捷键「Ctrl + Shift + P」打开命令窗口（all command window），输入「snippet」，通过候选栏中的选项进入目的语言的代码段设置文件。

## VSCode 中 snippet 的文法

```
// Place your snippets for C here. Each snippet is defined under a snippet name and has a prefix, body and 
 // description. The prefix is what is used to trigger the snippet and the body will be expanded and inserted. Possible variables are:
 // $1, $2 for tab stops, $0 for the final cursor position, and ${1:label}, ${2:another} for placeholders. Placeholders with the 
 // same ids are connected.
 // Example:
 "Print to console": {
    "prefix": "log",,
    "body": [
        "console.log('$1');",
        "$2"
    ],
    "description": "Log output to console"
}
```
## snippet 各参数介绍
snippet 由三部分组成：

- **prefix：**前缀，定义了 snippets 从 IntelliSense 中呼出的关键字;
- **body：** 主体，即模板的主体内容，其中每个字符串表示一行;
- **description：**说明，会在 IntelliSense 候选栏中出现。未定义的情况下直接显示对象名，上例中将会显示 Print to console。

其中 **body** 部分可以使用特殊结构来控制光标和要插入的文本。 支持的功能及其文法如下：

- **Tabstops：**制表符
用「Tabstops」可以让编辑器的指针在 snippet 内跳转。使用 $1，$2 etc. 指定光标位置。这些数字指定了Tabstops将被访问的顺序，特别地，$0表示最终光标位置。相同序号的「Tabstops」被链接在一起，将会同步更新，比如下列用于生成头文件封装的 snippet 被替换到编辑器上时，光标就将同时出现在所有$1位置。

```
"#ifndef $1"
"#define $1"
"#end // $1"
```
- **Placeholders：**占位符
「placeholder」是带有默认值的「Tabstops」，如 ${1：foo}。「placeholder」文本将被插入「Tabstops」位置，并在跳转时被全选，以方便修改。占位符还可以嵌套，例如 :
    > struct ${1:name_t} {\n\t$2\n};

- Variables：变量
使用$name或${name:default}可以插入变量的值。 当未设置变量时，将插入其缺省值或空字符串。 当varibale未知（即，其名称未定义）时，将插入变量的名称，并将其转换为「placeholder」。 可以使用以下「Variable」：

    - TM_SELECTED_TEXT：当前选定的文本或空字符串
    * TM_CURRENT_LINE：当前行的内容
    * TM_CURRENT_WORD：光标下的单词的内容或空字符串
    * TM_LINE_INDEX：基于零索引的行号
    * TM_LINE_NUMBER：基于一索引的行号
    * TM_FILENAME：当前文档的文件名
    * TM_DIRECTORY：当前文档的目录
    * TM_FILEPATH：当前文档的完整文件路径

## 引入文件的写入

知道了如上支持，我就开始着手写bootstrap的引入文件了：

``` JSON
"Print to console": {
		"prefix": "bootstrap",
		"body": [
			"<link rel=\"stylesheet\" href=\"http://cdn.static.runoob.com/libs/bootstrap/3.3.7/css/bootstrap.min.css\">",
			"<script src=\"http://cdn.static.runoob.com/libs/jquery/2.1.1/jquery.min.js\"></script>",
			"<script src=\"http://cdn.static.runoob.com/libs/bootstrap/3.3.7/js/bootstrap.min.js\"></script>"
		],
		"description": "This Jquery & BootStrap quate"
	}
```

值得注意的是：文件是以**json**的书写格式来的，而引号在其中要用**转义字符**，直接输入引号的话，是会产生错误的。

## json字符转义

官网也给出了 snippet 的 EBNF 范式的正则文法，注意，使用\（反斜杠）转义\$, ,, }和\。 

```
any ::= tabstop | placeholder | variable | text 
tabstop ::= '$' int | '${' int '}' 
placeholder ::= '${' int ':' any '}' 
variable ::= '$' var | '${' var }' | '${' var ':' any '}' 
var ::= [_a-zA-Z] [_a-zA-Z0-9]* 
int ::= [0-9]+ 
text ::= .* 
```
而在代码里我们的引号换成了：
> \"

## 结果
![jg1](https://ooo.0o0.ooo/2017/06/10/593bbf8b0d1a9.png)
![jg2](https://ooo.0o0.ooo/2017/06/10/593bbf8b24e7a.png)