---
layout:     post
title:      "如何在网页里插入全局新窗口打开超链接"
subtitle:   "在您的个人网站上加入全局全局的超链接都以_blank(新窗口打开)方式"
date:       2017-5-24
author:     "BoomXu"
header-img: "https://ooo.0o0.ooo/2017/05/27/592990249c072.jpg"
tags:
    - HTML
    - JavaScript
---


**很多站长朋友都遇到了自己写的文章或者网站里的一些页面打开的时候都是以_myself的方式打开的，这样很多阅读者就会遇到想看着自己写的教程而又点开操链接都是直接在本页面上打开，这样我们就直接用一点小技术可以解决**

## JavaScript代码实现所有超链接以_blank方式打开

``` HTML
<!--把一下js代码放到页面的尾部，也就是</body>标签的上面-->

<script language="JavaScript" type="text/javascript">
    <!--
        var a = document.getElementsByTagName("A");
        for(var i = 0;i < a.length;i++){
            a[i].target = "_blank";
        }
    //-->
</script>
```

### 完全可以把以上代码粘贴到HTML的body内就可以实现全局超链接了！！😀是不是很开心！！！！

## Jekyll博客模板可以以下：

1. 全局包括在首页上的所有连接，把代码复制到“_inclouds/footer.html”的末尾
2. 文章内上的所有连接，，把代码复制到“_layouts/post.html”（根据自己的模板来，默认是post）的末尾

-----

**以上就是现所有超链接以_blank方式打开的方法，希望小伙伴们能成功**

    
如果各位喜欢我的博客内容或者有什疑问可以在我的博客下方留言，或者给我发邮件！！

enjoy！

<br>

###后记：
   

刚刚发现了点问题，我想把我的文章页面弄成_blank方式打开，但是不包括下面这个“上一篇博文”和“下一篇博文”这样，但是一但加入到最底部整个文章页面的超链接都是会以_blank的方式打开网页，这怎么办呢，其实只需仔细分析下这个JS的运行原理：

``` HTML
<script language="JavaScript" type="text/javascript">
    <!--
        var a = document.getElementsByTagName("A");  //重要的是这一步
        for(var i = 0;i < a.length;i++){
            a[i].target = "_blank";
        }
    //-->
</script>
```

这个 **“getElementsByTagName()”** 到底是什么作用呢：

> 如果把特殊字符串 "*" 传递给 getElementsByTagName() 方法，它将返回文档中所有元素的列表，元素排列的顺序就是它们在文档中的顺序。

也就是说这段JS是以代码以上的超链接数来计算的。这样就好办了，把程序代码段放到按钮前面，搞定！

> 2017.06.02更新

**几天时间内一直在弄只修改{ {--content--} }变量内的a标签的target值，后来还是通过jQuery寻找div的a标签来实现了，在这里谢谢KeJun大神**

下面看看代码：

``` HTML
<script>
$(function(){
    $('article').find("#content").find('a').on('click',function(){
        $(this).attr("target","_blank")
    })
})
</script>
```

其中需要注意的是 jQuery的选择器，**“#”** 是选择ID **“.”**是选择class

总算是弄完了，下面开始弄自己搭建isso评论系统，还有好多事情做，这里end！
