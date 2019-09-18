---
title: css3 transform对普通元素的n多渲染影响
date: 2019-05-18 23:15:18
tags:
    - CSS
---

> 问题原因：项目中用到了vant框架，在dilog中想要弹出Actionsheet发现绝对定位是跟着dilog外层元素定位，此时的 `position:fixed` 无效，问题是因为dilog定位时用了transform，深究下问题原因，发现事情并不是这么简单！

一个普普通通的元素，如果应用了CSS3 transform变换，即便这个transform属性值不会改变其任何表面的变化（如scale(1), translate(0,0)），但是，实际上，对这些元素还是造成了很深远的影响。
# transform提升元素的垂直地位
当遭遇元素margin负值重叠的时候，如果没有static以外的position属性值的话，后面的元素是会覆盖前面的元素的。
`img src="mm1"><img src="mm2" style="margin-left:-60px;">`
在transform出现之前，这个规则一直很稳健；但是，自从transform降临，这个规则就变了。元素应用了transform属性之后，就会变得应用了position:relative一个尿性，原本应该被覆盖的元素会雄起，变成覆盖其他元素，修改为如下代码：
```HTML
<img src="mm1" style="-ms-transform:scale(1);transform:scale(1);">
<img src="mm2" style="margin-left:-60px;">
```
只要是支持transform元素的浏览器，包括IE9(-ms-), 都会提高普通元素的垂直地位，使其覆盖其他元素而不是被覆盖。

这种特性底层原理是层叠上下文，具体可参见“[深入理解CSS中的层叠上下文和层叠顺序](www.baidu.com)”一文。
# transform限制position:fixed的跟随效果
我们应该都知道，`position:fixed`可以让元素不跟随浏览器的滚动条滚动，而且这种跟随效果连它的兄弟们`position:relative/absolute`都限制不了。但是，真是一物降一物，`position:fixed`固定效果却被小小的`transform`给干掉了，直接降级变成`position:absolute`的蛋疼表现。

例如下面示意代码：

`<p style="transform:scale(1);"><img src="mm1.jpg"style="position:fixed;" /></p>`
结果，本来应该不跟着滚动条滚动的傲娇`fixed`元素，变成`absolute`一样的行为表现，归根结底就是父元素加了个小小的`transform`属性值。

注意，这个特性表现，目前只在Chrome浏览器/FireFox浏览器下有，IE浏览器，包括IE11, `fixed`还是`fixed`的表现。

# transform改变overflow对absolute元素的限制
在以前，`overflow`与`absolute`之间的限制规范内容大致是这样的：

`absolute`绝对定位元素，如果含有`overflow`不为`visible`的父级元素，同时，该父级元素以及到该绝对定位元素之间任何嵌套元素都没有`position`为非`static`属性的声明，则`overflow`对该`absolute`元素不起作用。

比方说如下示意代码：
```HTML
<p style="width:96px; height:96px; border:2px solid #beceeb; overflow:hidden;">
    <img src="mm1.jpg"style="position:absolute;" />
</p>
```

但是，一旦我们给`overflow`容器或者与图片有嵌套关系的子元素使用`transform`声明，估计`absolute`元素就要去领便当了！

无论是`overflow`容器还是嵌套子元素，只要有`transform`属性，就会`hidden`溢出的`absolute`元素。

# transform限制absolute的100%宽度大小
以前，我们设置`absolute`元素宽度100%, 则都会参照第一个非`static`值的`position`祖先元素计算，没有就`window`. 现在，需要把`transform`也考虑在内了。

结果，无论是IE9+，还是Chrome还是FireFox浏览器，所有绝对定位图片100%宽度，都是相对设置了`transform`的容器计算了，于是，上面的图片拉长到了西伯利亚；下面的图片被限制成了小胖墩。

`transform`对`absolute`宽度100%限制~
