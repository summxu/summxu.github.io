---
layout:     post
title:      "Linxu下安装Node.js"
subtitle:   "用Make编译的方式在Linux下安装Node.js"
date:       2017-06-26
author:     "BoomXu"
header-img: "https://ooo.0o0.ooo/2017/06/26/595080ef538a0.jpg"
tags:
    - Linux
    - Node.js
---



> 系统环境：Ubuntu 32位

# 下载Node.js

首先去[官网](http://nodejs.org)复制下Node.js的Source Code包地址

![1](https://ooo.0o0.ooo/2017/06/26/5950810f1484c.png)

然后用 **weget** 来下载编译包。

![2](https://ooo.0o0.ooo/2017/06/26/5950820c6c40c.jpg)

# 安装Node.js

先解压包
```
tar -xf node-v6.11.0.tar.gz
cd node-v6.11.0
```
配置并编译
```
./configure
make
```

![5](https://ooo.0o0.ooo/2017/06/26/5950812f9ba2a.jpg)

安装
```
sudo make install
```

# 检查安装
```
node -v
npm -v
```

![7](https://ooo.0o0.ooo/2017/06/26/595084d48efd7.jpg)

创建nodejs项目目录

> mkdir -p /usr/local/nodejs/

创建hello.js文件

> vi /usr/local/nodejs/hello.js

内容如下：
``` JavaScript
var http = require("http");
http.createServer(function(request, response) {
	response.writeHead(200, {
		"Content-Type" : "text/plain" // 输出类型
	});
	response.write("Hello BoomXu");// 页面输出
	response.end();
}).listen(8100); // 监听端口号
console.log("nodejs start listen 8100 port!");
```
开启nodejs服务

> node hello.js

![8](https://ooo.0o0.ooo/2017/06/26/595084d490492.jpg)

浏览器访问

http://127.0.0.1:8100/

![6](https://ooo.0o0.ooo/2017/06/26/595084d477f91.jpg)
