---
layout:     post
title:      "无需后台的集成式评论系统"
subtitle:   "介绍下国内外几个常见的评论系统,有发展也有落败"
date:       2017-6-02
author:     "BoomXu"
header-img: "https://ooo.0o0.ooo/2017/06/02/593188be04f4a.jpg"
tags:
    - JavaScript
---


> 然而图片和此文章没有一点毛关系，只是为了好看。

前言：作为一个博客系统，评论系统几乎是必备的。而很多静态博客像**Jekyll**几乎没有自己开发评论系统的可能性，所以我们就介绍下比较常用的几家评论系统，和自己去搭建评论系统。

# 国内评论系统

- [多说](http://duoshuo.com/)-------(已关闭)<br>
多说算是国内最好用的评论系统了，但是多说宣布与2017年6月1日关闭。
- [友言](http://www.uyan.cc/)<br>
不支持HTTPS，友言在评论管理方面比较全面，允许管理员自主灵活设置匿名评论功能，并且还支持数量最多的11种建站平台安装方式，但是友言在评论方式上表现较为不足。
- [网易云跟帖](https://gentie.163.com/)
总体而言，网易云跟贴在用户评价方面表现也较为全面，但是账号登录方式也较少，在评论管理方面最为不足。
- [畅言](http://changyan.kuaizhan.com/)
总体而言，畅言表现最为全面，且唯一支持管理员分权管理，但是畅言账号的登录方式较少，支持PC端安装的建站平台数量也较少。
本站也是正在使用畅言。

# 国外评论系统

- [Disqus](https://disqus.com/)

Disqus是国外的评论插件，自从多说关闭后也是现在很多站长都在使用的评论系统，其界面美观简单。但是，被墙了。。。

# 搭建自己的评论系统

## isso

Isso是一个轻量级的类似Disqus第三方评论系统，它允许匿名评论、注册评论、回复邮件通知以及自定义外观等功能。它的接口设计和Disqus高度相似，所以要集成这个评论系统只需要在Disqus接口上改几个单词，非常简单。

Isso是基于Python写的开源软件，你可以随意修改评论框外观。

![isso](https://ooo.0o0.ooo/2017/06/16/5943ed18afb33.png)

我在这里说下我看到的Docker容器的ISSO的安装方法，比较简单，不用去配置复杂的python环境。

用docker你首先要安装**docker环境** 和 **Compose** 。两个工具的安装在此不多赘述。。

1. 下载Docker镜像
    
    在docker上有人做出来了一个docker包，直接pull下来就可以，地址在这：<https://hub.docker.com/r/wonderfall/isso/>

2. 配置ISSO

    新建一个文件夹名为config，在里面新建一个配置文件isso.conf：

```
[general]
dbpath = /db/comments.db
host = https://www.BoomXu.com
[server]
listen = http://0.0.0.0:8080/
```

    下面是一个Compose配置文件：

```
version: '2'
services:
  isso:
    image: wonderfall/isso
    environment:
      - GID=1000
      - UID=1000
    volumes:
      - ./config:/config
      - ./db:/db
    ports:
      - "8080:8080"
```

    保存为docker-compose.yml然后执行：

```
dokcer-compose up -d
```

    搞定之后就可以通过8080端口的接口使用Isso评论系统了。

    启动后目录应该是这样的：

```
.
├── config
│   └── isso.conf
├── db
│   └── comments.db
└── docker-compose.yml

2 directories, 3 files
```
3. 设置

    Isso服务已经运行了，当然直接访问8080端口是没有什么界面的，只有一个API接口。接下来我们要在静态博客中集成这个评论系统。

    如果你使用的主题是纯HTML，那么嵌入下面两句即可：

```
<script data-isso="//comments.example.tld/"
        src="//comments.example.tld/js/embed.min.js"></script>

<section id="isso-thread"></section>
```

    如果你是Jade或者Ejs等模板引擎，那么复制一下Disqus的代码，替换为Isso即可，例如Jade格式。

    下面是Disqus的Jade模板：

```
if theme.disqus
    a#comments
    #disqus_thread
    script.
        var disqus_shortname = '#{theme.disqus}';
        var disqus_identifier = '#{page.path}';
        var disqus_title = '#{page.title}';
        var disqus_url = '#{config.url}/#{page.path}';
        (function() {
            var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
            dsq.src = '//' + disqus_shortname + '.disqus.com/embed.js';
            (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
        })();
    script(id='dsq-count-scr' src='//#{theme.disqus}.disqus.com/count.js' async)
```

    现在改写为Isso评论系统（类似）：

``` JavaScript
if theme.isso
    a#comments
    .isso-thread
    script.
        var isso-path = {short_name:"#{theme.isso}"};
        (function() {
            var isso = document.createElement('script');
            isso.type = 'text/javascript';ds.async = true;
            isso.src = (document.location.protocol == 'https:' ? 'https:' : 'http:') + '//example.com/isso/js/embed.js';
            ds.charset = 'UTF-8';
            (document.getElementsByTagName('head')[0] 
             || document.getElementsByTagName('body')[0]).appendChild(ds);
        })();
```

4. 评论计数

    如何在首页中显示文章计数？
    加入下面一句到页面中：

```
<a href="/my-uri.html#isso-thread">Comments</a>
```