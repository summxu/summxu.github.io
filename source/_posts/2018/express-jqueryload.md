---
layout:     post
title:      "Express + 模板引擎 + jQuery load 实现局部页面的异步加载"
subtitle:   "Express返回局部页面"
date:       2018-09-12
author:     "BoomXu"
tags:
    - Express
    - Node.js
---

> 项目匿名评论系统的局部异步加载服务的实现

## 问题需求

目前的论坛系统带有聊天室，在浏览帖子的同时可以随时聊天，这就意味着页面不能经常刷新，一是刷新就会重连服务器，导致一直断开重连的状态，二是刷新之后聊天记录消失，还要做数据的持久化，再者就是影响用户体验，所以最好的解决方案就是异步加载局部页面。前后端分离是挺好做，直接load加载本地页面就可以，但一和express结合起来就不知道该怎么办了。

![](http://p5oqx8gut.bkt.clouddn.com/18-9-12/5563645.jpg)

## 解决方案

其实仔细思考这个问题并不复杂，和前后端分离的模式是完全一模一样的，**只是把本地要加载的局部页面换成了服务器上渲染出来的局部页面** 。 这时就需要重新配置express的路由和 render 的页面分离，路由多了两条： 

``` JavaScript
.get('/main',(req,res) => {
  mongo.Post.find((err, postdata) => {
    if (err) return res.status(500)
    /* 对象属性抽离，解决template陷入递归 */
    /* mogon取出的对象不正常，转换一下 */
    var data = JSON.stringify(postdata)
    data = JSON.parse(data)
    var images = []
    data.forEach((element, a) => {
      images.push(element.images)
      delete element.images
    });
    res.render('./components/main.html', {
      post: data,
      images: images
    })
  })
})
.get('/sendpost',(req,res) => {
  res.render('./components/sendpost.html')
})
```

渲染出独立页面，这是各个页面的路径分配：

![](http://p5oqx8gut.bkt.clouddn.com/18-9-12/52866315.jpg)

这样两个在服务器路径中真实存在的页面出来了，通过浏览器也能访问，只不过没了样式：

![](http://p5oqx8gut.bkt.clouddn.com/18-9-12/99119243.jpg)

![](http://p5oqx8gut.bkt.clouddn.com/18-9-12/67452684.jpg)

在客户端上写上这么一句，就大功告成了！！

`$('.center .left').load("/main");`
`$('.center .left').load("/sendpost");`