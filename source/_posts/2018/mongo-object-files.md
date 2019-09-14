---
layout:     post
title:      "MongoDB的查找find方法取出不正经对象"
subtitle:   "对象格式化"
date:       2018-09-10
author:     "BoomXu"
tags:
    - MongoDB
    - 模板引擎
---

> 项目匿名评论系统的MongoDB操作

起因：用art-template来渲染前端页面的时候，数组里又包含了数组，这是模板引擎报了一个递归错误，貌似 **模板引擎不支持递归？** 想到的解决办法是对象的 **数组属性** 分离渲染，在删除对象属性时遇到了问题。

## delete 方法

原生 js 中支持 delete 方法，即删除对象里的某个属性，并且带有一个布尔类型的返回值，本事一个很简单的事情，放到普通代码中也很好实现，但是和MongoDB集合起来的时候，就出了问题。

原因就是 mongoDB find 出来的对象 **不正经**

![](http://p5oqx8gut.bkt.clouddn.com/18-9-10/77300539.jpg)

这里的id没有引号，原本js代码是不能执行，更不能直接删除对象的属性值，但是这里很奇怪的是，**js居然不会报错？？**而把这个对象格式直接复制到正常程序里却报错。

正确的解决办法就是转换一下 对象的类型：

```
/* 对象属性抽离，解决template陷入递归 */
/* mogon取出的对象不正常，转换一下 */
var data = JSON.stringify(postdata)
data = JSON.parse(data)
```

## 循环嵌套的异步

还遇到就是循环图片的时候，等循环结束要res。send（）图片数组，但是 fs.rename 是一个异步操作，要等到循环结束，可以这么操作：

```
if (req.files.pic.length > 1) {
  var c = req.files.pic.length
  req.files.pic.forEach(item => {
    var target_path = './upload/' + item.name
    fs.rename('./' + item.path, target_path, function (err) {
      if (err) throw err;
      postObj.images.push(target_path.replace('.',''))
      
      c--;
      if (c === 0 ){
        console.log(postObj.images)
        postObj.save((err, result) => {
        if (err) throw err
          res.send('success')
        })
      }
      // 删除临时文件夹文件, 
      fs.unlink('./' + item.path, function () {
        if (err) throw err;
      });
    });
  });
```

用计数器判断循环的次数，为 0 时执行代码。

听说用递归也可以