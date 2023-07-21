---
title: Webpack按需打包Lodash的几种方式
date: 2019-04-16 13:23:06
tags:
    - JavaScript
---
在数据操作时，Lodash 就是我的弹药库，不管遇到多复杂的数据结构都能用一些函数轻松拆解。

ES6 中也新增了诸多新的对象函数，一些简单的项目中 ES6 就足够使用了，但还是会有例外的情况引用了少数的 Lodash 函数。一个完整的 Lodash 库，即使是压缩后，现最新版本也有 71k 的体积。不能为了吃一口饭而买下一个饭店啊。

针对这个问题，其实已经有很多可选方案了。

# 函数模块
Lodash 中的每个函数在 NPM 都有一个单独的发布模块。`NPM: results for ‘lodash’`
假如你只需要使用`_.isEqual`，那么你只需要安装`lodash.isequal`模块，然后按以下方式引用。
```javascript
var isEqual = require('lodash.isequal')
// or ES6
import isEqual from 'lodash.isequal'
isEqual([1, 2, 3], [1, 2, 3]) // true
```
# 全路径引用
在你完整安装 Lodash 后，可以按`lodash/函数名`的格式单独引入需要的函数模块。
```javascript
var difference = require('lodash/difference')
// or ES6
import difference from 'lodash/difference'
difference([1, 2], [1, 3])  // [2]
```
# 使用插件优化
在简单场景下，以上两种方式足以解决问题。
而遇到复杂的数据对象时，我们不得不在一个文件中引入多个 Lodash 函数，这样就需要在文件中写多个`require`或`import`相关函数。
```javascript
import remove from 'lodash/remove'
import uniq from 'lodash/uniq'
import invokeMap from 'lodash/invokeMap'
import sortBy from 'lodash/sortBy'
// more...
```
正写到关键处却因为引入一个函数要拉到文件头部去定义引用而打乱了思路，很不爽！

于是我机智的到 Github 去搜索了webpack和lodash两个关键词的组合，排在首位的 lodash-webpack-plugin 就是为了解决这个问题而生。

使用时需要以下模块，其实除了前两个剩下的一般都已安装了：

`$ npm i -S lodash-webpack-plugin babel-plugin-lodash babel-core babel-loader babel-preset-es2015 webpack`
**配置：**
```javascript
webpack.config.js
var LodashModuleReplacementPlugin = require('lodash-webpack-plugin');
var webpack = require('webpack');
module.exports = {
  module: {
    loaders: [{
      loader: 'babel',
      test: /\.js$/,
      exclude: /node_modules/,
      query: {
        plugins: ['transform-runtime', 'lodash'],
        presets: ['es2015']
      }
    }]
  },
  plugins: [
    new LodashModuleReplacementPlugin,
    new webpack.optimize.OccurrenceOrderPlugin,
    new webpack.optimize.UglifyJsPlugin
  ]
}
```
其中`babel-plugin-lodash`的配置，也就是`plugins: ['lodash']`，并不是一定要在`loaders`中，也可以单独定义`babel`。
```javascript
webpack.config.js
var LodashModuleReplacementPlugin = require('lodash-webpack-plugin');
var webpack = require('webpack');
module.exports = {
  module: {
    loaders: [{
      loader: 'babel',
      test: /\.js$/,
      exclude: /node_modules/
    }]
  },
  babel: {
    presets: ['es2015'],
    plugins: ['transform-runtime', 'lodash']
  },
  plugins: [
    new LodashModuleReplacementPlugin,
    new webpack.optimize.OccurrenceOrderPlugin,
    new webpack.optimize.UglifyJsPlugin
  ]
}
```
又或者是`.babelrc`文件中。

以上工作完成了，在每个你需要使用 lodash 函数的文件中只需要引用一次 lodash，即可调用任意函数而不会造成完全打包。
```javascript
import _ from 'lodash'
_.add(1, 2)  // 打包时只会引入这一个函数模块
```
注意：必须要使用 ES2015 的模块引用方式才有效。

# End
以上即是我目前所知道的几种方式，如果哪位朋友有更好的方式（比如只需要全局引入一次），请一定分享与我！😋