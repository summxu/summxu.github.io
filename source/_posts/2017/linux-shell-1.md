---
layout:     post
title:      "Linux下Shell学习 --基础1"
subtitle:   "记录自己学习Shell的过程"
date:       2017-06-27
author:     "BoomXu"
header-img: "https://ooo.0o0.ooo/2017/06/27/5952449246b4e.jpg"
tags:
    - Linux
    - Shell
---



> 今天学习下Shell,发现非常简单，语法也很是随意，在此记录下学习过程。


# Shell介绍

## Shell和Shell脚本

Shell 是一个用 C 语言编写的程序，它是用户使用 Linux 的桥梁。Shell 既是一种命令语言，又是一种程序设计语言。
Shell 是指一种应用程序，这个应用程序提供了一个界面，用户通过这个界面访问操作系统内核的服务。
Ken Thompson 的 sh 是第一种 Unix Shell，Windows Explorer 是一个典型的图形界面 Shell。

Shell 脚本（shell script），是一种为 shell 编写的脚本程序。
业界所说的 shell 通常都是指 shell 脚本，但读者朋友要知道，shell 和 shell script 是两个不同的概念。
由于习惯的原因，简洁起见，本文出现的 "shell编程" 都是指 shell 脚本编程，不是指开发 shell 自身。

## Shell 环境

Shell 编程跟 java、php 编程一样，只要有一个能编写代码的文本编辑器和一个能解释执行的脚本解释器就可以了。
* Linux 的 Shell 种类众多，常见的有：
* Bourne Shell（/usr/bin/sh或/bin/sh）
* Bourne Again Shell（/bin/bash）
* C Shell（/usr/bin/csh）
* K Shell（/usr/bin/ksh）
* Shell for Root（/sbin/sh）
* ……

在一般情况下，人们并不区分 Bourne Shell 和 Bourne Again Shell，所以，像 #!/bin/sh，它同样也可以改为 #!/bin/bash。
#! 告诉系统其后路径所指定的程序即是解释此脚本文件的 Shell 程序。

# Hello world

```
#!/bin/bash
echo "Hello world"
```

# 变量及数组

```
#!/bin/bash
only="我是只读变量"
readonly only
#only="我不是只读变量"
del="我被删除了"
unset del
a='陈旭'
str="hello,${a}"

array=(陈旭 神还套 梁航宇)

echo "ni hao a wo shi  $a heihei "
echo $only
echo $del
echo $str
echo "字符串长度为：${#str}"
echo "提取字符串陈旭：${str:6:4}"

echo $array ${array[1]} ${array[2]}

echo "数组长度是：${#array[*]}"
echo "数组所有元素：${array[*]}"

```

# 参数传递

```
#!/bin/bash


echo "Shell 传递参数实例！";
echo "执行的文件名：$0";
echo "第一个参数为：$1";
echo "第二个参数为：$2";
echo "第三个参数为：$3";

echo "参数个数 $#"
echo "参数字符串 $*"
echo "运行ID号 $$"
```

# 运算符表达式
```
#!/bin/bash
a=10
b=22

add=`expr $a + $b`
quyu=`expr $b % $a`
cheng=`expr $a \* $b`

echo "A+B=：$add"
echo "取余：$quyu"
echo "A乘B：$cheng"
if [ $a == $b ]
then
echo "A大于B"
fi
echo "B大于A"

```

## 关系运算符

关系运算符只支持数字，不支持字符串，除非字符串的值是数字。
下表列出了常用的关系运算符，假定变量 a 为 10，变量 b 为 20：

运算符 | 说明 | 举例
----|----|---
-eq | 检测两个数是否相等，相等返回 true。 | [ $a -eq $b ] 返回 false。
-ne | 检测两个数是否相等，不相等返回 true。 | [ $a -ne $b ] 返回 true。
-gt | 检测左边的数是否大于右边的，如果是，则返回 true。 | [ $a -gt $b ] 返回 false。
-lt | 检测左边的数是否小于右边的，如果是，则返回 true。 | [ $a -lt $b ] 返回 true。
-ge | 检测左边的数是否大于等于右边的，如果是，则返回 true。 | [ $a -ge $b ] 返回 false。
-le | 检测左边的数是否小于等于右边的，如果是，则返回 true。 | [ $a -le $b ] 返回 true。