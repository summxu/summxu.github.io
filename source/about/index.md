---
layout: page      # 必须
title: BoomXu  # 必须，页面名称
description: 资深前端小小白..       # 页面二级标题，描述性文字
comments: false     # 禁用评论，可选，默认开启
reward: false       # 禁用打赏，可选，默认开启
---


## Timeline

@card{

在`timeline`模块中，你的 5 号标题`#####`和六号标题`######`将被“征用”，用作时间线上的标记点：

```css
@timeline{

##### 2016

@item{
###### 11月6日

为 Card theme 添加 page layout。

}

}
```

`@item`中多行内容可以换行输入，目前不允许隔行：

```css
@timeline{

##### 2016

@item{
###### 11月6日

第一行 
第二行 /* ok */

}

@item{
###### 11月6日

第一行

第二行 /* error */

}

}
```

}

@timeline{

##### 2016

@item{
###### 11月6日

为 Card theme 添加 page layout。
加快绿化空间好看

}

@item{
###### 10月31日

本地化多说。

}

@item{
###### 10月24日

为 Indigo 主题创建 Card 分支。

}

##### 2015

@item{
###### 2月24日

发布 Indigo 主题到 hexo.io。

}

@item{
###### 1月22日

创建 Indigo 主题。

}

}
