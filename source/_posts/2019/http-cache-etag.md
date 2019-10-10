---
title: HTTP缓存-ETag
date: 2019-08-10 20:44:11
tags:
    - HTTP
---

> 最近在学习网站性能优化相关的内容，关于网站优化点特别多而HTTP缓存也是比较重要的一部分，于是今天就着重看下HTTP缓存相关的内容加深下对此相关知识的理解和认识。自己动手通过简单的服务，看看其中的过程。

## ETag验证缓存的响应

在本地通过 express 启了一个非常简单的个服务，具体如下

```javascript
// app.js
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('hello http')
})

app.listen(3000, () => {
  console.log('The server is running at http://127.0.0.1:3000/')
})
```

但是仔细看却发现，第一次进入页面**http://127.0.0.1:3000/**时，Status为200而再次刷新发现Status却是304

![](https://upload-images.jianshu.io/upload_images/8532055-e14290fe5a1aaa4b.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

![](https://upload-images.jianshu.io/upload_images/8532055-f19b1eb12e8d8cc4.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

仔细对比发现

1. 第一次请求时候请求参数中并没有 If-None-Match 字段但是却有个Pragma；同时在请求的Response中有一个 **ETag: W/"a-QFZ79AprHeNlMfPMKXyEUV+lyOg"**字段。
2. 刷新页面后再次请求在请求头中却有个 **If-None-Match: W/"a-QFZ79AprHeNlMfPMKXyEUV+lyOg" **，If-None-Match 的值和第一次请求的ETag的值相同。

经过查询才了解原理浏览器会根据HTTP请求的ETag验证请求的资源是否发生了改变，如果它未发生变化，服务器将返回“304 Not Modified”响应，并且资源从浏览器缓存中读取，这样就不必再次下载请求。

由此看来整个的过程就是下面这样：

1. 如果缓存中有ETag 令牌，客户端请求时会自动在“If-None-Match” HTTP 请求标头内提供 ETag 令牌。
2. 服务器根据当前资源核对令牌，验证是否发生变化，将验证结果通知给客户端，客户端根据结果看看是否需要从缓存中读取还是发送资源请求。

补充一个很直白的 [TCP协议的三次握手](https://github.com/jawil/blog/issues/14) 的理解。

为了验证查证的结果，我又添加一个请求处理。这个过程是，客户端明确返回一个ETag, 但是这里每次请求的的返回值都不相同，这里简单的使用了个**etag++**。

```javascript
// app.js
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('hello http');
})


// 验证ETag
let etag = 0;
app.get('/test', (req, res) => {
  etag++;
  res.set('ETag', etag);
  res.send('ETag');
})

app.listen(3000, () => {
  console.log('The server is running at http://127.0.0.1:3000/')
})
```

![](https://upload-images.jianshu.io/upload_images/8532055-71230f0ad656f455.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

查看下 /test 地址的请求结果，会发现If-None-Match 的值和 Response中的 ETag值每次都不相同，并且是 浏览器会将每次的 ETag 值都缓存起来在下次请求的时候发送给服务器。这样一来，每次服务器每次校验的值都是不相同的，所以这种就没有做缓存，因此每次请求 /test 地址都是 200 的状态。