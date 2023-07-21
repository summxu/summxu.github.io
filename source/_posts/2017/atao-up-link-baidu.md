---
layout:     post
title:      "通过JS代码提交连接到百度"
subtitle:   "通过JS代码提交连接到百度来让百度更快的收录你的文章！"
date:       2017-06-11
author:     "BoomXu"
header-img: "https://ooo.0o0.ooo/2017/06/11/593cec37f048b.jpg"
tags:
    - JavaScript
---



为了加快百度对博客文章的收录，站长们最好提交网站文章的连接到百度，百度提供了四种方式：

# 主动推送(实时)

百度官方说这是最好的推送方式，但是这种静态博客不支持，像PHP的博客wordprass可以只用这种方式，百度也给出了几个提交例子：

![initiative](https://ooo.0o0.ooo/2017/06/11/593ceec783f4e.png)

# sitemap

以站点地图的形式来提交网站，站点地图中可以填入你网站所有的连接，我认为这适用与一些比较大的网站且是第一次提交连接。

![sitmao](https://ooo.0o0.ooo/2017/06/11/593ceec78e57a.png)

# 手动提交

把连接逐条填写到网站中提交，如果连接过多的情况下挺麻烦的。

![manual](https://ooo.0o0.ooo/2017/06/11/593ceec79dd59.png)

# 自动推送

自动推送是百度站长平台为提高站点新增网页发现速度推出的工具，安装自动推送JS代码的网页，在页面被访问时，页面URL将立即被推送给百度。

![aoto](https://ooo.0o0.ooo/2017/06/11/593ceec79f0c8.png)

其代码如下：

``` HTML
<!-- 百度实时网页推送 --> 
<script>
(function(){
    var bp = document.createElement('script');
    var curProtocol = window.location.protocol.split(':')[0];
    if (curProtocol === 'https'){
   bp.src = 'https://zz.bdstatic.com/linksubmit/push.js';
  }
  else{
  bp.src = 'http://push.zhanzhang.baidu.com/push.js';
  }
    var s = document.getElementsByTagName("script")[0];
    s.parentNode.insertBefore(bp, s);
})();
</script>
<!-- 百度实时网页推送 END--> 
```

本站是采用自动提交的JavaScript代码来提交网站连接。