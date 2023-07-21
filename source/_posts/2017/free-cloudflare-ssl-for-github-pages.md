---
layout:     post
title:      "为自定义的域名添加免费SSL"
subtitle:   "Github Pages再也不怕没有SSL的小绿标了."
date:       2017-5-24
author:     "BoomXu"
header-img: "https://ooo.0o0.ooo/2017/05/26/592836b3d58e3.jpg"
tags:
    - 技术分享
---


 原文连接 [为自定义域名的GitHub Pages添加SSL 完整方案](https://www.yicodes.com/2016/12/04/free-cloudflare-ssl-for-custom-domain/)


+ 简单总结一下这个过程：

1. 在 github page 的设置中填入 custom domain

2. 去你的域名注册商更改 dns server 成 cloudflare 所提供的

3. 在 cloudflare 设置 A record ，参照

4. 在 cloudflare 配置 crypto ，选择 flexiable （这一步要一段时间才能起效）

5. 在 cloudflare 配置 page rule ，一个是 always use https ，一个是 redirect http 到 https 每一步的操作可能需要 5-30 分钟才能起效

### 为什么使用Cloudflare提供的免费SSL
  
  
收费的SSL服务总是比免费的更加周到，一般收费的SSL都会提供端到端的加密。但是价格不菲，对于个人博客来说，这是一笔不必要的开销。我只是需要看到网站地址栏有绿色的锁头，那就证明我们的网站相对安全了。
  
  
此外，使用https之后，谷歌、百度等搜索排名权值（PR等）也会有相对提升。
还有其他的一些，例如Cloudflare还提供免费的CDN和缓存技术，让浏览者有更好的体验~~

> 好了，说了那么多，直接看教程~~

### 创建CloudFlare帐户，并添加网站

**首先你已经有自己的自定义域名的GitHub Pages** ，我的 GitHub Pages cname文件写的是 yicodes.com

> 实现目标： 当访客输入 yicodes.com 强制跳转使用https，访问wwww 也会跳转到https://www.yicodes.com

+ 如果你还没有Cloudflare账号，[点击注册](https://www.cloudflare.com/a/sign-up)

+ 登陆后，点击[这里](https://www.cloudflare.com/a/login) 增加你的域名，如下图，输入你的域名，例如 **yicodes.com **并点击 **Begin Scan**

> 注意不要写WWW前缀，大约60秒即可完成域名解析扫描。完成后点击 **Continue Setup** 继续下一步

![](https://www.yicodes.com/img/gpssl/github-page-ssl-1.png)

+ 你看到DNS记录（包括子域）列表之后，按照下图提示设置后，其中cname是为了重定向www准备的，点击 **Continue** 下一步

![](https://www.yicodes.com/img/gpssl/dnsrecord.png)

+ 选择免费计划，然后下一步~

![](https://www.yicodes.com/img/gpssl/plan.png)

+ 到你域名控制面板修改cloudflare给出的域名服务器，我这里以 Godaddy 为例

![](https://www.yicodes.com/img/gpssl/updata-nameservers.png)
![](https://www.yicodes.com/img/gpssl/updata-nameservers1.png)
![](https://www.yicodes.com/img/gpssl/updata-nameservers2.png)

注：官方说明，**域名服务器修改最长需要72小时生效** ，用了两个域名测试，大约需要 5~30 分钟，看到 Status: Active 即可

![](https://www.yicodes.com/img/gpssl/status.png)

### 设置SSL

+ 点击 crypto 菜单 , 然后设置 Flexible SSL ，如下图

![](https://www.yicodes.com/img/gpssl/flexiblessl.png)

+ 添加www重定向到https://www.yicodes.com

![](https://www.yicodes.com/img/gpssl/add301.png)

+ 添加自动重定向到 SSL页面

![](https://www.yicodes.com/img/gpssl/forcessl.png)

添加SSL的教程就此完成，**一般需要5~30分钟生效！！！** 如果你有疑问，欢迎到我博客留言
