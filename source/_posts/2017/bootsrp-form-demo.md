---
layout:     post
title:      "初学BootStrap --Form Demo"
subtitle:   "记录下自己的BootStrap样式和JQuery的学习道路。"
date:       2017-06-12
author:     "BoomXu"
header-img: "https://ooo.0o0.ooo/2017/06/09/593ab6fe6b6e2.jpg"
tags:
    - jQuery
    - BootStrap
---


> 根据MOOC上的教程做了一个小小的demo。也从现在开始创建一个可以直接预览的效果。

## [点此预览](https://yuyu147.github.io/Bootstrap/form&viewport&demo.html)

``` HTML
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>表单和viewport例子</title>
    <link rel="stylesheet" href="bootstrap.min.css">
    <script src="jquery.min.js"></script>
    <script src="bootstrap.min.js"></script>
</head>
<body>
    <p class="text-center">请注册您的账号：</p>
    <form>
    <div class="form-group has-error">
        <input type="text" class="form-control" placeholder="请输入您的手机号码">
    </div>
    <div class="form-group has-success">
        <label for="">请选择您的城市：</label>
        <select class="form-control">
            <option value="">北京</option>
            <option value="">上海</option>
            <option value="">济南</option>
            <option value="">成都</option>
            <option value="">重庆</option>
        </select>
    </div>
    <div class="form-group">
        <input type="button" class="" value="确定注册">
        <input type="button" class="btn btn-block" value="确定注册">
        <input type="button" class="btn-success active" value="确定注册">
        <input type="button" class="btn-primary" disabled="disabled" value="确定注册">
        <input type="button" class="btn-info" value="确定注册">
        <input type="button" class="btn-warning" value="确定注册">
        <input type="button" class="btn-danger btn-sm" value="确定注册">
        <input type="button" class="btn-default btn-lg" value="确定注册">
        <input type="button" class="btn-link" value="确定注册">
    </div>
    <a href="" class="btn btn-default btn-lg">这是用a标签的按钮效果</a>
    </form>
</body>
</html>
```