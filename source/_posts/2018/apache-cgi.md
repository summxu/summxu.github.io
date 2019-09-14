---
layout:     post
title:      "Apahce的CGI操作 - Python"
subtitle:   "用Apache解决Python作为后台web接口的解决方法"
date:       2018-04-13
author:     "BoomXu"
header-img: "http://p5oqx8gut.bkt.clouddn.com/18-4-13/13824140.jpg"
tags:
    - Linux
    - Apache
    - Python
---

> 最近在研究Podcast泛博客的订阅地址，在国内的几大Podcast平台基本都提供RSS订阅源地址，但还是有很多专辑没有被分享订阅地址出来，碰巧最近在研究Python，打算用Python做爬虫，分析出来各大节目的广播地址并生成订阅源头。如果用前台去调用后台接口大部分的方式都是用PHP和Apache的方法，为了不想再涉及到PHP，下面就用这种古老的CGI的方式来解决。

# 什么是 CGI 

CGI 是Web 服务器运行时外部程序的规范,按CGI 编写的程序可以扩展服务器功能。CGI 应用程序能与浏览器进行交互,还可通过数据库API 与数据库服务器等外部数据源进行通信,从数据库服务器中获取数据。格式化为HTML文档后，发送给浏览器，也可以将从浏览器获得的数据放到数据库中。几乎所有服务器都支持CGI,可用任何语言编写CGI,包括流行的C、C ++、VB 和Delphi 等。CGI 分为标准CGI 和间接CGI两种。标准CGI 使用命令行参数或环境变量表示服务器的详细请求，服务器与浏览器通信采用标准输入输出方式。间接CGI 又称缓冲CGI,在CGI 程序和CGI 接口之间插入一个缓冲程序，缓冲程序与CGI 接口间用标准输入输出进行通信。

# 怎么用Apache来调用指定CGI

其实调用原理很简单,无非就是配置apache的cgi支持、设置cgi的运行目录和运行cgi的后缀

## 配置对CGI的支持

`LoadModule cgi_module modules/mod_cgi.so`

## 定义CGI的目录

```
ScriptAlias /cgi-bin/ "/var/www/cgi-bin/"

#
# "/var/www/cgi-bin" should be changed to whatever your ScriptAliased
# CGI directory exists, if you have that configured.
#
<Directory "/var/www/cgi-bin">
    Options Indexes FollowSymLinks MultiViews +ExecCGI
    AllowOverride None
    Options None
    Order allow,deny
    Allow from all
</Directory>
```

## 定义CGI支持的文件后缀

`AddHandler cgi-script .cgi .py .sh`

# 编写Python脚本的CGI  
```
#!/usr/bin/env python
# -*- coding: UTF-8 -*-

import cgi
import cgitb
print "Content-type:text/html"
print

form = cgi.FieldStorage()
name = form.getvalue('name')
print name

```

这里要注意的是 apache 调用cgi的时候必须要输出的这一句 ：
```
print "Content-type:text/html"
print
```
这个应该是固定格式，具体的原因的话没再深入研究