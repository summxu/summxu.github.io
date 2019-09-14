---
layout:     post
title:      "CSS控制背景图片亮度"
subtitle:   "通过CSS来改变background亮度实现突出文字效果。"
date:       2017-06-15
author:     "BoomXu"
header-img: "https://ooo.0o0.ooo/2017/06/15/59422525066f9.jpg"
tags:
    - CSS
    - HTML
---



> 不美观引发的技术思考即实现。。

# 前言

打开博客，细细的看了看自己写的小东西，虽说不好但却是自己一点一点的小经验。但是，种感觉有什么怪怪的。。

![1](https://ooo.0o0.ooo/2017/06/15/59422d135296a.jpg)

仔细看看：

![2](https://ooo.0o0.ooo/2017/06/15/59422d171a019.png)

**卧槽。。**发现文字颜色为什么和图片融为一体了？？？？我是瞎了吗这么久都没看出来！！！！**得赶紧想想办法**

# 想法：opacity

首先想到的就是，**调暗图片颜色**

记得可以使用：**css3的opacity:x**调整图片透明度，上面弄上全黑背景，因为用的jekyll，所以直接改代码。。。

找到**post.html** 看了看头部发现代码如下：
```
header.intro-header{
        background-image: url('{% if page.header-img %}{ { page.header-img } }{% else %}{ { site.header-img } }{% endif %}');
}
```
很简单，直接就一个 _background-image_ 然后直接加上覆盖上一个css试试。
```
header.intro-header{
        background-image: url('{% if page.header-img %}{ { page.header-img } }{% else %}{ { site.header-img } }{% endif %}');
        opacity: 1;
}
```
![3](https://ooo.0o0.ooo/2017/06/15/59422d12af472.jpg)

发现可以透明了，但是整个文字也透明了，这不符合要求啊。。。百度了下，原来用 _opacity_ 是不符合要求的。

> 实现透明的css方法通常有以下3种方式，以下是不透明度都为80%的写法

> - css3的opacity:x，x 的取值从 0 到 1，如opacity: 0.8
> - css3的rgba(red, green, blue, alpha)，alpha的取值从 0 到 1，如rgba(255,255,255,0.8)
> - IE专属滤镜 filter:Alpha(opacity=x)，x 的取值从 0 到 100，如filter:Alpha(opacity=80)

> 使用说明：设置opacity元素的所有后代元素会随着一起具有透明性，一般用于调整图片或者模块的整体不透明度..那么使用opacity实现《背景透明，文字不透明》是**不可取**的。

**opacity **代码示例：

```
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>css3的rgba</title>
<style>
*{
    padding: 0;
    margin: 0;
}
body{
    padding: 50px;
    background: url(img/bg.png) 0 0 repeat;
}
.demo{
  padding: 25px;
  background-color:#000000;/* IE6和部分IE7内核的浏览器(如QQ浏览器)下颜色被覆盖 */
  background-color:rgba(0,0,0,0.2); /* IE6和部分IE7内核的浏览器(如QQ浏览器)会读懂，但解析为透明 */
}
.demo p{
    color: #FFFFFF;
}
</style>
</head>
<body>    

<div class="demo">
    <p>背景透明，文字也透明</p>
</div>

</html>
```

# rgba

> 很奇葩的是，IE6和部分IE7内核的浏览器(如QQ浏览器)会读懂rgba，解析后颜色为透明，其实应该是null.

管不了这么多了，IE这种被时代抛弃的东西，我才不考虑。.

看了下示例代码：
```
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>css3的rgba</title>
<style>
*{
    padding: 0;
    margin: 0;
}
body{
    padding: 50px;
    background: url(img/bg.png) 0 0 repeat;
}
.demo{
  padding: 25px;
  background-color:#000000;/* IE6和部分IE7内核的浏览器(如QQ浏览器)下颜色被覆盖 */
  background-color:rgba(0,0,0,0.2); /* IE6和部分IE7内核的浏览器(如QQ浏览器)会读懂，但解析为透明 */
}
.demo p{
    color: #FFFFFF;
}
</style>
</head>
<body>    

<div class="demo">
    <p>背景透明，文字也透明</p>
</div>

</html>
```

加到我的Jekyll post 里看看效果：

![4](https://ooo.0o0.ooo/2017/06/15/59422d1318cd9.jpg)

```
<style type="text/css">
        .mengban{
            background-color:#000000;
            background-color:rgba(0,0,0,0.2); 
        }
        header.intro-header{
        background-image: url('{% if page.header-img %}{ { page.header-img } }{% else %}{ { site.header-img } }{% endif %}');
        opacity: 1;
        }
</style>
```

可以了，但是还是有点白 ， 调整下参数。。完美解决。。

![5](https://ooo.0o0.ooo/2017/06/15/59422d16cc5cc.png)

不过既然写到这了，在说下IE的:

# IE专属滤镜 filter:Alpha

使用说明：IE浏览器专属，问题多多，本文以设置背景透明为例子，如下：

1. 仅支持IE6、7、8、9，在IE10版本被废除

2. 在IE6、7中，需要激活IE的haslayout属性(如：*zoom:1或者*overflow:hidden)，让它读懂filter:Alpha

3. 在IE6、7、8中，设置了filter:Alpha的元素，父元素设置position:static(默认属性)，其子元素为相对定位，可让子元素不透明

示例代码如下：

```
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>opacity</title>
<style>
*{
    padding: 0;
    margin: 0;
}
body{
    padding: 50px;
    background: url(img/bg.png) 0 0 repeat;
}
.demo{
  padding: 25px;
  background: #000000;
  filter:Alpha(opacity=50);/* 只支持IE6、7、8、9 */
  position:static; /* IE6、7、8只能设置position:static(默认属性) ，否则会导致子元素继承Alpha值 */
  *zoom:1; /* 激活IE6、7的haslayout属性，让它读懂Alpha */
}
.demo p{
    color: #FFFFFF;
    position: relative;/* 设置子元素为相对定位，可让子元素不继承Alpha值，保证字体颜色不透明 */
}      

</style>
</head>
<body>    

<div class="demo">
    <p>背景透明，文字不透明</p>
</div>
```


> 本文部分文章出处：<http://peunzhang.cnblogs.com/>