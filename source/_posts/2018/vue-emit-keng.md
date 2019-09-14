---
layout:     post
title:      "Vue.$emit Promise 回调后的深坑"
subtitle:   ""
date:       2018-09-25
author:     "BoomXu"
tags:
    - Vue
    - JavaScript
---

> 有个登录需求，是 login.vue 属于 App.vue 的子组件，默认路由页面是进入 login.vue 因为你一开始需要验证登录用户，又要通过登录用户来进行
