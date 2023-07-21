---
layout:     post
title:      "精简实用的VScode前端扩展"
subtitle:   "用VScode Exdtened打造最好最实用的前端开发工具！"
date:       2017-06-08
author:     "BoomXu"
header-img: "https://ooo.0o0.ooo/2017/06/08/5939158e0d98e.jpg"
tags:
    - 技术分享
---


# 前言
vscode是一款跨平台的代码编辑器，她轻量、美观、一致、功能完整，自带完美git支持，非常适合前端同学使用。下面总结下我对于这个软件的使用技巧，希望对大家有帮助。
# 快捷键
注意：当下列快捷键不起作用时，请考虑是否是其他软件已经占用了快捷键，如输入法、聊天软件等

vscode内置了emmet，使用tab键可快速编写html/css等，具体请查询emmet语法说明  

快捷键 | 备注
----|---
Ctrl + N | 新建文件
Ctrl + S | 保存文件
Ctrl + F | 在当前文件内查找
Ctrl + H | 在当前文件内替换
Ctrl + Shift + F | 在文件内查找
Ctrl + Shift + H | 在文件内替换
Ctrl + Tab | 切换文档
Ctrl + PgUp/PgDn | 向前/向后切换文档
Ctrl + 1/2/3 | 切换分栏
Ctrl + P | 转到文件
Ctrl + P | 转到文件
Ctrl + G | 转到行，输入行号，转到该行号处
Ctrl + 鼠标滚轮 | 缩放编辑器显示比例
Ctrl + F1 | 在浏览器中打开当前正在编辑的html文件，这个功能需要插件支持，安装插件：View In Browser
F12 | 转到定义，可跳转到变量定义处，定义较为复杂时，会找不到
F1 | 打开命令输入框
Shift + Alt + F | 格式化代码
Ctrl + Alt + Up/Down/Left/Right | 按区域选中代码，并编辑
Alt + Up/Down | 移动光标所在行或者光标选中代码的位置
Alt + Left/Right | 前进或后退操作历史，这个历史不是编辑历史（Ctrl + Z/Y操作的是编辑历史），它包含如：第一步，光标在12行6列处；第二步，光标在20行10列处；第三步，打开另外一个文件。
Shift + Alt + Up/Down | 复制光标所在行至上一行或下一行
Alt + 光标选中 | 光标多选，同时编辑


# 扩展/插件

vscode支持扩展，官方商店里有很多扩展，网址：https://marketplace.visualstu...
安装方式：点击编辑器左侧的【扩展】按钮，在搜索框中输入你想要安装的插件，点击安装即可。

推荐插件

链接 | 名称 | 备注
---|----|---
[查看](https://marketplace.visualstudio.com/items?itemName=christian-kohler.path-intellisense) | Path Intellisense | 在编辑器中输入路径时，自动补全
[查看](https://marketplace.visualstudio.com/items?itemName=christian-kohler.path-intellisense) | Auto Close Tag | 自动补全html标签，如输入<a>将自动补全</a>
[查看](https://marketplace.visualstudio.com/items?itemName=formulahendry.auto-rename-tag) | Auto Rename Tag | 自动重命名html标签，如修改为，将自动修改结尾标签</a>为</b>
[查看](https://marketplace.visualstudio.com/items?itemName=humao.rest-client) | REST Client | 在编辑器中发送http请求，可直接得到结果，测试接口时很有用处，用法请看插件详情
[查看](https://marketplace.visualstudio.com/items?itemName=mrcrowl.easy-less) | Easy LESS | 自动编译less文件（文件保存后自动变为为同名.css文件），如果你只是想用less，而又不想配置grunt等工具来使用它时，请使用这个插件，他绝对是更效率的
[查看](https://marketplace.visualstudio.com/items?itemName=qinjia.view-in-browser) | View In Browser | 按Ctrl + F1在浏览器中打开正在编辑的html文件
[查看](https://marketplace.visualstudio.com/items?itemName=robertohuertasm.vscode-icons) | vscode-icons | 为文件添加炫酷的图标，图标种类很全，包括各种配置文件、常见语言、常见js框架等，你值得拥有；安装后，需要使用管理员权限启动vscode，并打开命令输入框(F1或者Ctrl+Shift+P)执行Icons Enable

如有任何问题可以在下方留言或者给我邮件。