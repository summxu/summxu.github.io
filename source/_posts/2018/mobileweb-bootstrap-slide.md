---
layout:     post
title:      "Bootstrap响应式轮播图问题"
subtitle:   "Js控制响应式"
date:       2018-08-25
author:     "BoomXu"
tags:
    - JavaScript
    - Bootstrap
---

> bootstrap 原生轮播图，因为轮播图是图片，所以为了做响应式就要麻烦一些，因为要涉及到图片的大小还有小屏幕的显示比例，把这次遇到的问题重新捋一下 。

# 两站单独设计

因为PC端的轮播图是定高的，而且图片尺寸又大，但是移动端是百分比可缩小的，图片是小图，所以只能是两站单独设计，这是两站单独的CSS:

```
.pc_img {
  display: block;
  height: 400px;
  background: no-repeat center;
  background-size: cover;
}
.m_img {
  display: block;
  width: 100%;
}
.m_img img{
  display: block;
  width: 100%;
}
```

这样HTML的结构就可以这么放(用了模板引擎之后)，并且用了响应式工具使两者实现不同屏幕尺寸单独显示，不会同时显示出来。。

```
<div class="item { { $index ? '' : 'active'} }">
  <a class="m_img hidden-lg hidden-md hidden-sm" alt="{ {$index} } slide"><img src="{ {$value} }" alt="" srcset=""></a>
  <a class="pc_img hidden-xs" alt="{ {$index} } slide" style="background-image: url('{ {$value} }');"></a>
</div>
```

这样虽说是视觉上看不出来有什么问题，但是在请求中两者都会同时请求，加载的图片太多，而且只是小屏幕的话，会多加载大屏幕中的大图片。。。

为了优化 不得不用js判断访问的屏幕大小。这样一系列的问题就来了。

# 屏幕大小判断并单独加载

因为要单独判断当前屏幕的大小来指定到底要加载哪种图片，什么结构，又要做响应式，就会涉及到以下两点问题：

1. 判断页面大小，请求页面数据，渲染模板引擎
2. 响应式实时判断页面大小，而不重复发送请求

难点在第二点，我们只需要页面在需要请求数据的时候才不得不请求（如果现在是大屏幕，只请求大图片，如果是小屏幕，之请求小图片。如果从大屏幕响应式到了小屏幕，这时不得不再去求情一次）

做响应式，实时判断页面大小，又不用每次改变大小时去实时发送请。这时我们可以在特定的时候来请求数据，并把数据缓存下来，这里用的方法是绑定到 **window** 上一个全局对象的属性，并且用 **回调函数** 把全局对象的属性返回，具体代码如下： 

```
/* 判断访问来源 ， 以便使pc和m都加载 */
$(document).ready(function () {
  function getData(callback) {
    /* 判断 window.data 是不是有缓存数据 */
    if (window.data) {
      callback && callback(window.data)
    }else {
      $.ajax({
        type: "GET",
        url: "./data.json",
        dataType: "json",
        // async: false,
        success: function (response) {
          window.data = response
          callback && callback(window.data)
        }
      });
    }
  }
  function render() {
    /* 这里通过回调函数获取传递过来的数据，是为了每次调用 render 的时候不ajax */
    getData(function (response) { 
      var data = {}
      var iSMobile = window.innerWidth < 750 ? true : false
      if (iSMobile) {
        data.type = 'm'
        data.content = response.m
        var html = template('carousel-content',data)
        $('.banner .carousel-inner').html(html)
        var html = template('carousel-indicators',data)
        $('.banner .carousel-indicators').html(html)
      }else{
        data.type = 'pc'
        data.content = response.pc
        var html = template('carousel-content',data)
        $('.banner .carousel-inner').html(html)
        var html = template('carousel-indicators',data)
        $('.banner .carousel-indicators').html(html)
      }  
    })
  }
  /* 这里要做响应式，所以要把数据渗透成全局对象 window.data */
  $(window).on('resize',function () { 
    render()
    /* 通过js主动触发事件 */
  }).trigger('resize',function (){
    render()
  })
```

这里要注意一点，就是当数据请求和没请求的两种状态下，都要进行回调函数

```
success: function (response) {
  window.data = response
  callback && callback(window.data)
}
```

最后用 jQuery 的 trigger 方法来主动调用一次 window.onresize 就不用手动调用 render() 了。

## 实现移动端滑动功能

```
/* 加载轮播图滑动 控制 prev 和 next */
  var startX = 0
  var endX = 0
  isMove = false
  $('#carousel-id').on('touchstart', function (e) {
    startX = e.originalEvent.touches[0].clientX;
  }).on('touchmove',function(e){
    isMove = true
    /* originalEvent 是原生事件的属性。 */
    endX = e.originalEvent.touches[0].clientX;
  }).on('touchend',function(e){
    lr = endX - startX
    tmp = Math.abs( endX - startX)
    if (isMove && tmp > 50) {
      if (lr < 0) {
        $('.carousel').carousel('next')
        console.log('next');
      }else {
        $('.carousel').carousel('prev')
        console.log('prev')
      }
    }
  });
});
```

# 关于模板引擎

模板引擎是个好东西，在获取到数据绑定到HTML页面上的时候，用DOM方式来操作确实有点难受，而且还弄的十分混乱，特别是当数据比较多的时候。这时候模板引擎能更友好的将数据渲染到页面上，甚至模板引擎本身提供了简单好用的一些方法

这里使用的模板引擎是 [Arttemplate](https://aui.github.io/art-template/zh-cn/docs/index.html)
其他的模板引擎的使用方法都大同小异，基本的使用方法大概是这样：
js:

```
<!-- 渲染html结构 ，放到 script 标签中-->
var html = template('arttemplate',{value:data})
<!-- 把渲染好的HTML放到指定的容器中 -->
document.getEmementById('content').innerHTML(html)
```

因为script 的 type 只要是不为 **text/javascript** 就不会别解析成js语法 所以很多时候方便存一些有结构型的数据到script标签中，模板引擎也是基于这个。 但是模板引擎 使用 { {  } } 里面的内容是 **可以直接写 js 代码的**
html:

```
<script id="carousel-content" type="text/x-arttemplate">
  { { if $data.type == 'm' } }
    { { each $data.content } }
      <div class="item { { $index ? '' : 'active'} }">
        <a class="m_img hidden-lg hidden-md hidden-sm" alt="{ {$index} } slide"><img src="{ {$value} }" alt="" srcset=""></a>
      </div>
    { { /each } }
    { { else } }
      { { each $data.content } }
        <div class="item { { $index ? '' : 'active'} }">
          <a class="pc_img hidden-xs" alt="{ {$index} } slide" style="background-image: url('{ {$value} }');"></a>
        </div>
      { { /each } }
  { { /if } }
</script>
<script id="carousel-indicators" type="text/x-arttemplate">
  { { each $data.content } }
    <li data-target="#carousel-id" data-slide-to="{ {$index} }" class="active"></li>
  { { /each } }
</script>
```

这样页面就能方便的处理数据了。 