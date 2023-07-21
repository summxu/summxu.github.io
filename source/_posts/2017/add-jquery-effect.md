---
layout:     post
title:      "通过jQuery添加网页的动画效果"
subtitle:   "用jQuery美化了下博客文章页的标题个tags,主要运用到jQuery效果."
date:       2017-06-15
author:     "BoomXu"
header-img: "https://ooo.0o0.ooo/2017/06/15/5942a94c2114a.jpg"
tags:
    - jQuery
---



> 看了下同学的Hexo博客，Next的动态主题简直帅炸，不服气，我的Jekyll也可以做到。刚会点jQuery正好顺便研究一下动画效果。随便弄弄也花费了**2个小时**，不过**学到了很多**。

# jQuery 效果函数

由于新手所以只弄了jQuery提供的默认动画（感觉已经很好看了）。查阅相关资料：

方法 | 描述
---|---
animate() | 对被选元素应用“自定义”的动画
clearQueue() | 对被选元素移除所有排队的函数（仍未运行的）
delay() | 对被选元素的所有排队函数（仍未运行）设置延迟
dequeue() | 运行被选元素的下一个排队函数
fadeIn() | 逐渐改变被选元素的不透明度，从隐藏到可见
fadeOut() | 逐渐改变被选元素的不透明度，从可见到隐藏
fadeTo() | 把被选元素逐渐改变至给定的不透明度
hide() | 隐藏被选的元素
queue() | 显示被选元素的排队函数
show() | 显示被选的元素
slideDown() | 通过调整高度来滑动显示被选元素
slideToggle() | 对被选元素进行滑动隐藏和滑动显示的切换
slideUp() | 通过调整高度来滑动隐藏被选元素
stop() | 停止在被选元素上运行动画
toggle() | 对被选元素进行隐藏和显示的切换

这是jQuery提供的效果函数，发现可以用到的默认的其实只有两个：**fadeIn() 、 slideToggle()**

# 代码

因为**fadeIn() 、 slideToggle()** 函数都是在元素为隐藏的情况下才能显示效果，所以要先在样式里把div给隐藏掉，为了不出错，我先写了一个小例子：

[可以点击此处预览下效果](https://yuyu147.github.io/jQuery/jQuery%E5%8A%A8%E7%94%BB%E6%95%88%E6%9E%9C.html)

``` HTML
<style>
        .demo{
            display:none;
        }
        .demo1{
            display:none;
        }
    </style>
    <script>
        $(document).ready(function () {
            $(".demo1").slideToggle(1500);
            $("#effect1").slideToggle(1500);
            $("#effect2").fadeIn("slow");
            $("#effect3").slideDown(1500);
        });
        $(function(){ 
        $("#ok").click(function(){ 
            $("#effect4").animate({left: "500px"},3000) 
            .animate({height:"200px"},1000); 
        });
}) 
    </script>
</head>
<body>
    <div id="effect1" class="demo">
        <p>我是逐渐下拉的效果</p>
        <p>或许两行才能更好的体现</p>
    </div>
    <div id="effect2" class="demo">
        <p>我是淡入淡出的效果</p>
        <p>或许两行才能更好的体现</p>
    </div>
    <div id="effect3" class="demo">
        <p>我是向下滑动的效果</p>
        <p>或许两行才能更好的体现</p>
    </div>
    <div id="effect4">
        <p>我是向上滑动的效果</p>
        <p>或许两行才能更好的体现</p>
    </div>
    <div class="demo1">
        <p>我没有id</p>
    </div>
    <input id="ok" type="button" value="点我试试">
</body>
```

然后写到了Jekyll _post里:

```
<style type="text/css">
    .tags{
        display: none;
    }
    .title{
        display: none;
    }
    .effect{
        display: none;
    }
</style>

<div class="tags">
    {% for tag in page.tags %}
    <a class="tag" href="{ { site.baseurl } }/tags/#{ { tag } }" title="{ { tag } }">{ { tag } }</a>
    {% endfor %}
</div>
    <h1 class="title">{ { page.title } }</h1>
    <div class="effect">
    {% comment %}
    {% endcomment %}
    {% comment %} if page.subtitle {% endcomment %}
    <h2 class="subheading">{ { page.subtitle } }</h2>
    {% comment %} endif {% endcomment %}
    <span class="meta">Posted by {% if page.author %}{ { page.author } }{% else %}{ { site.title } }{% endif %} on { { page.date | date: "%B %-d, %Y" } }</span>
</div>
                        
```

# 遇到的问题

其实代码很简单，但是我缺花了近一小时的时间去寻找错误（一开始代码写上不能正常显示）。先是**更换本地jQuery**然后有换**远程jQuery引用** ，却一直没有成功。

最后发现，**jQuery要放到页面的头部？？？？** 为什么和我学的不太一样？？？不是引用文件放到哪都可以吗？

![nm](https://ooo.0o0.ooo/2017/06/15/5942ae2292a2a.jpg)