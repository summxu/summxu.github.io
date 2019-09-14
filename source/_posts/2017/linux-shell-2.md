---
layout:     post
title:      "Linux下Shell学习 --基础2"
subtitle:   "记录自己学习Shell的过程"
date:       2017-06-27
author:     "BoomXu"
header-img: "https://ooo.0o0.ooo/2017/06/27/5952449246b4e.jpg"
tags:
    - Linux
    - Shell
---



> 今天学习下Shell,发现非常简单，语法也很是随意，在此记录下学习过程。

# echo

echo 命令也很简单，在这里记下几个不常用到的。

## 显示命令执行结果

```
echo `date`
```
## 显示不换行
```
#!/bin/sh
echo -e "OK! \c" # -e 开启转义 \c 不换行
echo "It is a test"
```
## 显示换行
```
echo -e "OK! \n" # -e 开启转义
echo "It it a test"
```

# test

## 文件测试

参数 | 说明
---|---
-e 文件名 | 如果文件存在则为真
-r 文件名 | 如果文件存在且可读则为真
-w 文件名 | 如果文件存在且可写则为真
-x 文件名 | 如果文件存在且可执行则为真
-s 文件名 | 如果文件存在且至少有一个字符则为真
-d 文件名 | 如果文件存在且为目录则为真
-f 文件名 | 如果文件存在且为普通文件则为真
-c 文件名 | 如果文件存在且为字符型特殊文件则为真
-b 文件名 | 如果文件存在且为块特殊文件则为真

# 分支语法

```
#!/bin/bash

num1=$[2*3]
num2=$[1+5]
if test $[num1] -eq $[num2]
then
    echo '两个数字相等!'
else
    echo '两个数字不相等!'
fi

```

# 循环语法

Shell的循环方式比较多，一开始不太好用， 我选择了基本和C语言方法相同的语法，现在理解是基本用法都差不多。

## C语言方式

```
#!/bin/bash

num1=$[2*3]
num2=$[1+5]
if test $[num1] -eq $[num2]
then
    echo '两个数字相等!'
else
    echo '两个数字不相等!'
fi

echo `expr 100 % 3`

#100以内可以被3整除
for ((i=1;i<100;i++))
do
if [ `expr $i % 3` == 0 ]
then
echo $i
fi
done

```

## for in 

```
for var in item1 item2 ... itemN
do
    command1
    command2
    ...
    commandN
done
```
例子：
```
for loop in 1 2 3 4 5
do
    echo "The value is: $loop"
done
```

## while 语句
```
while condition
do
    command
done
```

# case
```
echo '输入 1 到 4 之间的数字:'
echo '你输入的数字为:'
read aNum
case $aNum in
    1)  echo '你选择了 1'
    ;;
    2)  echo '你选择了 2'
    ;;
    3)  echo '你选择了 3'
    ;;
    4)  echo '你选择了 4'
    ;;
    *)  echo '你没有输入 1 到 4 之间的数字'
    ;;
esac
```

# 跳出循环

## break命令

break命令允许跳出所有循环（终止执行后面的所有循环）。
下面的例子中，脚本进入死循环直至用户输入数字大于5。要跳出这个循环，返回到shell提示符下，需要使用break命令。
```
#!/bin/bash
while :
do
    echo -n "输入 1 到 5 之间的数字:"
    read aNum
    case $aNum in
        1|2|3|4|5) echo "你输入的数字为 $aNum!"
        ;;
        *) echo "你输入的数字不是 1 到 5 之间的! 游戏结束"
            break
        ;;
    esac
done
```
## continue
continue命令与break命令类似，只有一点差别，它不会跳出所有循环，仅仅跳出当前循环。
对上面的例子进行修改：
```
#!/bin/bash
while :
do
    echo -n "输入 1 到 5 之间的数字: "
    read aNum
    case $aNum in
        1|2|3|4|5) echo "你输入的数字为 $aNum!"
        ;;
        *) echo "你输入的数字不是 1 到 5 之间的!"
            continue
            echo "游戏结束"
        ;;
    esac
done
```

# 函数

```
#!/bin/bash

funWithReturn(){
    echo "这个函数会对输入的两个数字进行相加运算..."
    echo "输入第一个数字: "
    read aNum
    echo "输入第二个数字: "
    read anotherNum
    echo "两个数字分别为 $aNum 和 $anotherNum !"
    return $(($aNum+$anotherNum))
}
funWithReturn
echo "输入的两个数字之和为 $? !"

```

## 函数参数
```
#!/bin/bash

funWithParam(){
    echo "第一个参数为 $1 !"
    echo "第二个参数为 $2 !"
    echo "第十个参数为 $10 !"
    echo "第十个参数为 ${10} !"
    echo "第十一个参数为 ${11} !"
    echo "参数总数有 $# 个!"
    echo "作为一个字符串输出所有参数 $* !"
}
funWithParam 1 2 3 4 5 6 7 8 9 34 73
```

参数处理 | 说明
-----|---
$# | 传递到脚本的参数个数
$* | 以一个单字符串显示所有向脚本传递的参数
$$ | 脚本运行的当前进程ID号
$! | 后台运行的最后一个进程的ID号
$@ | 与$*相同，但是使用时加引号，并在引号中返回每个参数。
$- | 显示Shell使用的当前选项，与set命令功能相同。
$? | 显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误。

# 求素数DEMO

```
#!/bin/bash
#求100以内的所有素数

for((i=2;i<100;i++))
do
    k=1
    for((j=2;j<i;j++))
    do
        if((`expr $i % $j` == 0))
        then
        k=0
        break
        fi
    done
    if [ $k == 1 ]
    then
    echo $i
    fi
done
```