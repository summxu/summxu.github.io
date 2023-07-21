---
layout:     post
title:      "HTTPS自动跳转JS代码"
subtitle:   "HTTP到HTTPS用js代码跳转来实现全站301指向"
date:       2017-06-11
author:     "BoomXu"
header-img: ""
tags:
    - JavaScript
---


# 前言

> 因为某种原因，本站的博客部署在了 Coding Pages 。其很大原因主要是为了兼顾百度的收录问题（GitHub停止了百度收录），但是却遇到了重重困难。

## Coding pages开启了https

![https enble](https://ooo.0o0.ooo/2017/06/11/593ce3f1da679.png)

 如果启用了强制HTTPS访问，百度抓取就会 出现一下错误：

![error](https://ooo.0o0.ooo/2017/06/11/593ce44b5bedb.png)
    
上面提示异常信息：**连接重定向次数超过5次的上限**。这里就有两个疑问：

1. 我是hrtps的站点，抓取的时候确实http？
2. 我确实做了强行301到https（刚才coding上那个选项），但是为什么说重定向超过5次？

我感觉这次抓取的结果可以重三个方面分析：

- 我是hrtps的站点，抓取的时候确实http？
    - 首先这个原因，暂且认为百度抓取只会抓HTTP的页面**并不会抓HTTPS**，这个原因直接导致了抓取http出现有跳转的情况。(但是想了想可能又不是这样，百度搜索引擎是不可能连一个HTTPS都不抓的。不过从抓取结果上来看，它确实只是抓取的http)
- 为什么说重定向超过5次？
    1. 有可能是因为Coding.Coding的强制跳转到HTTPS是不是就跳转了一次，还中间做了其他的页面？
    2. 百度抓取问题。

所以说这个问题至今都很奇怪，除了百度，其他所有的搜索引擎都可以成功去抓取。

索性关掉CodingPages的301功能来用JavaScript代码试下。

# 代码实现

## 检测来自百度蜘蛛并跳转

``` HTML
    <!-- 判断百度爬虫 -->
    <script>
        $(function(){
        var s=document.referrer;    //获取来源地址
        var targetProtocol = "https:";
        if(s.indexOf("baidu")>0){
            if (window.location.protocol != targetProtocol){
            window.location.href = targetProtocol + window.location.href.substring(window.location.protocol.length);  //跳转到HTTPs
            }
        }
        });
    </script>
    <!-- 判断百度爬虫END -->
```

这段代码其实是检测百度抓取而不是其他引擎或者用户正常访问来做的跳转，其中：
> document.referrer  //获取来源地址
> window.location.href  //只身跳转，可传递搜索引擎权重并不会引起各种浏览器屏蔽
> indexOf()  // 方法可返回某个指定的字符串值在字符串中首次出现的位置。

## 直接HTTP到https跳转
``` html
<script>
        var targetProtocol = "https:";
            if (window.location.protocol != targetProtocol){
            window.location.href = targetProtocol + window.location.href.substring(window.location.protocol.length);  //跳转到HTTPs
            }
    </script>
```

> window.location.protocol  //检测当前访问是不是HTTP

# 结果

![rithg](https://ooo.0o0.ooo/2017/06/11/593ce44b5bedb.png)

百度能正常抓取了，看样子是http的页面，不知道为什么，真的是好乱，而且JS跳转真的是不入直接301.唉，为了百度的索引。

更新下，已经有页面抓取到了。

![end](https://ooo.0o0.ooo/2017/06/11/593cea972b222.png)

先这么滴把。