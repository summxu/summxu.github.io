---
layout:     post
title:      "初次尝试Webpack构建前端"
subtitle:   "工程化高效化前端工具"
date:       2018-04-21
author:     "BoomXu"
header-img: "http://p5oqx8gut.bkt.clouddn.com/18-4-21/19914311.jpg"
tags:
    - JavaScript
    - Node.js
---

> 最近一直研究前端工程化和更高效的构建工具，网上很多人都说Node.js能帮助前端开发，今天终于了解了一下，原有的js,jquery,css,html都是基础，想要做出完整的前端页面还是很复杂，而且文件也比较多。初次接触了下Webpack，这真的是前端离不开的工程构建工具.

*首先在用这些东西之前最好是了解以下:*
- css基础
- JavaScript基础
- ES6
- css预处理器(less,sass,stulus)
- 对象编程逻辑

# 什么是Webpack

> 本质上，webpack 是一个现代 JavaScript 应用程序的静态模块打包器(module bundler)。当 webpack 处理应用程序时，它会递归地构建一个依赖关系图(dependency graph)，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个 bundle。

webpack就是把网页资源文件打包成一个js,在此之前你编程上可以用多种语言和多种工具,高效开发.在运行上网页速度加载也很快,减少了大量的http请求.

webpack有极高的配置度和丰富的插件,loader以及webhttp调试.

# Webpack 安装环境

> 在这之前要说下 Node.js 和 NPM

## Node.js

Node.js 是一个基于 Chrome V8 引擎的 JavaScript 运行环境。 
Node.js 使用了一个事件驱动、非阻塞式 I/O 的模型，使其轻量又高效。 

## NPM

> npm 为你和你的团队打开了连接整个 JavaScript 天才世界的一扇大门。它是世界上最大的软件注册表，每星期大约有 30 亿次的下载量，包含超过 600000 个 包（package） （即，代码模块）。来自各大洲的开源软件开发者使用 npm 互相分享和借鉴。包的结构使您能够轻松跟踪依赖项和版本。

NPM包管理工具异常强大,他可以把很多个包的东西都关联起来.我们搭建本次webpack使用了以下的各种包:

1. "cross-env": "^5.1.4",
2. "css-loader": "^0.28.11",
3. "file-loader": "^1.1.11",
4. "html-webpack-plugin": "^3.2.0",
5. "style-loader": "^0.21.0",
6. "stylus": "^0.54.5",
7. "stylus-loader": "^3.0.2",
8. "url-loader": "^1.0.1",
9. "vue": "^2.5.16",
10. "vue-loader": "^14.2.2",
11. "vue-template-compiler": "^2.5.16",
12. "webpack": "^4.6.0",
13. "webpack-dev-server": "^3.1.3"

**Webpack** 在我们这里也是一种包 其安装方法如下:

`npm install webpack`

## NPM package.json文件

> 每个项目的根目录下面，一般都有一个package.json文件，定义了这个项目所需要的各种模块，以及项目的配置信息（比如名称、版本、许可证等元数据）。npm install命令根据这个配置文件，自动下载所需的模块，也就是配置项目所需的运行和开发环境。

其实就是一个json格式的包管理文件,以及这种命令操作配置,这个文件配置很灵活,我们也要经常添加各种配置.

以下是我的文件配置 : 
``` JSON
{
  "name": "vuewebpacke",
  "version": "1.0.0",
  "description": "test",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "cross-env NODE_ENV=production webpack --config webpack.config.js",
    "dev": "cross-env NODE_ENV=development webpack-dev-server --mode development --config webpack.config.js"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "cross-env": "^5.1.4",
    "css-loader": "^0.28.11",
    "file-loader": "^1.1.11",
    "html-webpack-plugin": "^3.2.0",
    "style-loader": "^0.21.0",
    "stylus": "^0.54.5",
    "stylus-loader": "^3.0.2",
    "url-loader": "^1.0.1",
    "vue": "^2.5.16",
    "vue-loader": "^14.2.2",
    "vue-template-compiler": "^2.5.16",
    "webpack": "^4.6.0",
    "webpack-dev-server": "^3.1.3"
  },
  "devDependencies": {
    "webpack-cli": "^2.0.14"
  }
}

```

![](http://p5oqx8gut.bkt.clouddn.com/18-4-21/290804.jpg)

这就是没有配置loader造成的对.vue文件不识别

# Webpack 目录结构

## Webpack.config.js

> package-lock.json是当 node_modules 或 package.json 发生变化时自动生成的文件。这个文件主要功能是确定当前安装的包的依赖，以便后续重新安装的时候生成相同的依赖，而忽略项目开发过程中有些依赖已经发生的更新。

这是默认文件,我们不必动这个

## webpack.config.js

这是webpack的配置文件,他是在运行的时候加载的一种文件格式,属于node.js的模块
我们用了es6来编辑和配置这个文件里的各个参数和值,下面是文件内容,我们一一来解释:

```
//首先载入用的到的node.js库文件 
const path = require('path')      //path node.js 自带的模块用来生成出口文件
const HTMLPlugin = require('html-webpack-plugin')     //html-webpack-plugin 是生成html入口页面的插件
const webpack = require('webpack')

const isDev = process.env.NODE_ENV === 'development'

const config = {
  target: 'web',
    entry: path.join(__dirname,'src/index.js'),     //入口文件
    output:{
        filename: 'bundle.js',
        path: path.join(__dirname,'dist')     //__dirname是node.js全局变量,是文件目录
    },      //生成出口文件
    module: {
        rules: [
            {
                test: /\.vue$/,    //正则表达式
                loader: 'vue-loader'
            },
            {
                test: /\.css$/,
                use: [
                    'style-loader',
                    'css-loader'
                ]
            },
            {
              test: /\.styl/,
              use:[
                'style-loader',
                'css-loader',
                'stylus-loader'
              ]
            },
            {
                test: /\.(gif|png|jpeg|jpg|svg)$/,
                use: [
                  {
                    loader: 'url-loader',
                    options: {
                      limit: 1024,    //url-loader配置,图片小于1024使用base64编码
                      name: '[name]-xu.[ext]'     //输出图片名字格式
                    }
                  }
                ]
            }
        ]
    },      //各种loader声明方式
    plugins: [
      new webpack.DefinePlugin({
        'process.env':{
          NODE_ENV: isDev ? '"development"' : '"production"'
        }
      }),     //动态指定打包模式development or production
      new HTMLPlugin()
    ]       //插件数组,用到的插件都需要用new声明
}

if (isDev) {
  config.devtool = '#cheap-module-eval-source-map'
  config.devServer = {
    port: 8000,
    host: '0.0.0.0',
    overlay: {
      errors: true,
    },
    hot: true
  }
  config.plugins.push(
    new webpack.HotModuleReplacementPlugin(),
    new webpack.NoEmitOnErrorsPlugin()
  )
}

module.exports = config     //配置加载对象化

```

在此贴上webpack更具体的用法

[www.webpackjs.com/concepts](https://www.webpackjs.com/concepts/)

# 运行webpack

`npm run build or dev`

```
"build": "cross-env NODE_ENV=production webpack --config webpack.config.js",
"dev": "cross-env NODE_ENV=development webpack-dev-server --mode development --config webpack.config.js"
```
<hr>

之后还要在继续看看 **stylus,es6,vue**