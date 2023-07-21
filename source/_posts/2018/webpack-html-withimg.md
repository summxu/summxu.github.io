---
layout:     post
title:      "Webpack CSS分类打包和html img不打包配置"
subtitle:   "Webpack"
date:       2018-08-24
author:     "BoomXu"
tags:
    - JavaScript
---

> 介绍两个webpack插件的使用


# 问题还原

1.  在webpack打包之后的文件，往往都会打包到bundle.js中。但是为了目录的维护性，通常要把CSS单独分离出来这时需要

## extract-text-webpack-plugin

`npm install extract-text-webpack-plugin --save-dev`

配置上首先引入插件，然后再创建多个新的 extract 对象，并且在 plugins 引用他们

``` JavaScript
var Extract = require('extract-text-webpack-plugin')
var extractCss = new Extract('css/base.css')
var extractLess = new Extract('css/[name].css')
var extractScss = new Extract('css/[name].css')
var extractStyl = new Extract('css/[name].css')

plugins: [
  extractCss,
  extractLess,
  extractScss,
  extractStyl
],
```
下一步就是修改原本的 rules loader规则
``` JavaScript
{ test: /\.css$/, use: extractCss.extract( ['css-loader']) }, // 处理 CSS 文件的 loader
{ test: /\.less$/, use: extractLess.extract(['css-loader', 'less-loader']) }, // 处理 less 文件的 loader
{ test: /\.scss$/, use: extractScss.extract(['css-loader', 'sass-loader']) }, // 处理 scss 文件的 loader
{ test: /\.styl$/, use: extractStyl.extract(['css-loader', 'stylus-loader']) }, // 处理 stylus 文件的 loader
{ test: /\.(jpg|png|gif|bmp|jpeg)$/, use: 'url-loader?limit=1024&name=[name].[ext]&outputPath=./images&publicPath=../images'}, // 处理 图片路径的 loader
```
[https://www.npmjs.com/package/extract-text-webpack-plugin](https://www.npmjs.com/package/extract-text-webpack-plugin)

虽然上面引入了 url-loader 来解析图片文件，但是html内的img：src图片并不能通过此方式正常解析，原因是因为，webpack不能正确找到引入的图片，解决方式有两种。、
1.  让webpack知道引入的图片路径
`<img src="${ require('..assets/logo.png') }">`
2.  借助loader

## html-withimg-loader
`npm install html-withimg-loader --save`

在rule 中加入

`{ test: /\.(htm|html)$/i, loader: 'html-withimg-loader'} //处理html img 中的图片`

完事！

[https://github.com/wzsxyz/html-withimg-loader](https://github.com/wzsxyz/html-withimg-loader)
