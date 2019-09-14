---
layout:     post
title:      "博客大改之Ajax、Cookie、PHP"
subtitle:   "前台后台的数据传递"
date:       2018-3-25
author:     "BoomXu"
header-img: "http://p5oqx8gut.bkt.clouddn.com/18-3-26/8618257.jpg"
tags:
    - JavaScript
    - PHP
---

> 最近一个星期内，我一直在修改我的博客，直到今天，才算是基本完型。中间修改了很多地方，也学到了很多东西，在这篇博客里就一起回顾下这次博客修改遇到的各种问题吧！

* 因为中间隔得天数实在是太多，我也没有时间当时就去整理一番，现在再总结的话，中间肯定会有很多遗漏的地方， 所以说以后每天遇到的新的技术问题一定要及时记录下来。

## Jekyll

在一开始我先根据Jekyll的内部功能来添加日记，其中主要运用了Jekyll的文章分类功能 **categories** 我一直以为Jekyll 没有这种功能， 而且在网上查找资料都说得要这么实现很复杂，还有说要修改Jekyll的gem文章遍历的插件，其实并没有这么复杂，只需在文章头部添加文章标签就行了，关于Jekyll 的更多用法，以下这篇博客已经说得很完美了。

[http://ju.outofmemory.cn/entry/149459](http://ju.outofmemory.cn/entry/149459)

## Cookie

我使用了Jquery的cookie插件，其使用方法很简单，试用cookie的原因主要是不用每篇日记或者说是每次打开的时候都会进行密码验证。一开始想的是直接把密码从服务端取出来然后在本地判断，这样还省去了每次都向服务器端发送局的麻烦。但是这样行不通，因为密码一旦做本地判断，就不安全。

### 1.写入Cookie 

```
$.cookie("写入的cookie名","写入的cookie值",{

expires:7, //含义：有效期，单位：天，可写值：数字、日期对象，默认：如果不写或写null则浏览器关闭后cookie删除

path:"/", //路径，默认是创建该cookie的页面路径

domain:"地址", //域名属性，默认是创建该cookie的页面域名

secure:true //如果为true，那么此cookie的传输会要求一个安全协议，例如https

});
```
### 2.读取cookie
jQuery的读取cookie很简单：

`$.cookie("读取的cookie名")`
### 3.删除cookie

`写法：$.cookie("删除的cookie名",null) //参数null：代表删除此cookie`

cookie一开始一直有一个问题，就是每一页的验证好像都是一个cookie，找到资料发现是因为没有配置 **path** 如果要想全站cookie，只要加上 **path:"/"** 就好了。下面再继续说下Ajax。
## Ajax

其实在开始研究学习ajax之前，我感觉这是一种很麻烦动东西，但是仔细研究发现ajax并不是这么难，只是有时候会出现莫名其妙的问题。看看代码和写法：

``` JavaScript
$.ajax({
        type: "GET",
        url: "http://192.168.1.112/check.php",
        data: {pass : $("#pass").val()},  //发送出去的数据
        dataType : "json", //返回的数据格式
        async : false,  //同步请求
        success: function (data) {
            if(data.status == 1){
               alert(data.status);
            }else{
               alert(data.status);
            }
        },  // 成功后之执行
        error: function(){
            alert('产生了莫名其妙的错误！');
        } // 失败后执行
    });
```
~~ajax普通的传递,这几个参数还是很 简单的。第一个遇到的问题就是：~~

2018.8.28 回来看到之前的这个博客那时候的技术还是太浅了， 当时只涉及到jQuery的ajax封装函数，还不知道JavaScript的ajax的请求过程。
现在想想，ajax就是创建一个xmlhttprequest对象，再使用open建立连接，然后发送请求直至接受请求。
这时可能涉及到请求过程中的4种状态码：

0：初始化，XMLHttpRequest对象还没有完成初始化
1：载入，XMLHttpRequest对象开始发送请求
2：载入完成，XMLHttpRequest对象的请求发送完成
3：解析，XMLHttpRequest对象开始读取服务器的响应
4：完成，XMLHttpRequest对象读取服务器响应结束

```
var xhr = new XMLHtttpRequest ()
xhr.open('GET','./data.json',true)
xhr.send()
xhr.load(){}
```

- 跨域请求问题

    本地测试是可以正常传输，但是Github上是不支持任何服务器可执行的脚本的，所以我必须用另一个可执行后台的服务器来进行密码验证，这样就要涉及到跨域请求。我在网络上收集了各种解决方法，其中有用 **jasonp、加header头** 还有各种很复杂的方法，但是在这其中header头是最简单的。只需要在后台PHP里加上这么一句：`header('Access-Control-Allow-Origin:*');`

- 异步请求问题
    ajax中有一个参数就是 `async : false` 就是是否异步请求，默认是异步的，这个意思就是 **ajax运行的时候能同时运行其他的东西** ~~就像是多线程一样 但是在实际用的时候，我发现跨域用异步请求的话会很大几率出现请求不成功的情况，我不知道是为什么。~~

## PHP后台接口

PHP我之前尝试着做过，但是一直没有怎么学，也没有实际的项目能用上php，现在可以了，我完全可以用PHP做一个后端判断的接口。PHP代码很简单，但是有这么两点注意的地方：

    1.做接口一定要加上 `error_reporting(E_ERROR);`我之前见过别人的接口就是加上了这个，这句话的意思是不让PHP输出错误提示，有时候输出来错误提示返回出来的数据就不是标准的格式了。
    2.就是跨域请求的header头。

```
<?php  
    // /*验密码码是否正确*/ 
    error_reporting(E_ERROR);
    header('Access-Control-Allow-Origin:*');
    $code = trim($_GET['pass']);//接收前端传来的数据

    $raw_success = array('status' => 1 , 'word' => $code);  
    $res_success = json_encode($raw_success);  
    
    $raw_fail = array('status' => 0);
    $res_fail = json_encode($raw_fail);  
    
    if ($code == "想什么呢") {  
        echo $res_success;  
    } else {  
        echo $res_fail;  
    }
?>  
```