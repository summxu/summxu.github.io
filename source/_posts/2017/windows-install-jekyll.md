---
layout:     post
title:      "如何在Windows下安装Jekyll"
subtitle:   "一步一步带你在windows下安装Jekyll,附Windows一键安装脚本"
date:       2017-06-14
author:     "BoomXu"
header-img: "https://ooo.0o0.ooo/2017/06/14/59414d0871f46.jpg"
tags:
    - 技术分享
---



# windows环境下安装

# 1. 安装 Ruby

**官网：**<http://rubyinstaller.org/downloads/>

**测试：** Ruby是否安装成功，执行命令ruby -v

注意：勾选 “Add Ruby executables to your PATH”，安装路径不能包含空格

# 2. 安装 DevKit

**官网：**<http://rubyinstaller.org/downloads/>

**步骤：** 在命令窗口下切换到安装目录，并执行以下命令

```
ruby dk.rb init
ruby dk.rb install
```
**测试：** gem是否安装成功，执行命令gem -v

注意：Ruby与DevKit版本要对应

# 3. 安装 Jekyll

**步骤：** 在命令窗口下执行以下命令

```
//更换gem源
gem sources --remove https://rubygems.org/
gem sources -a https://ruby.taobao.org/

//查看gem源
gem sources -l

//更新gem
gem update --system

//安装jekyll
gem install jekyll
```

# 4. 安装 Python 相关环境
## a. 安装 Python

**官网：**<http://www.python.org/download/>

**测试：** python是否安装成功，执行命令python -V

注意：下载Python 2安装，如果安装Python 3 可能不会正常工作。安装后添加环境变量。

## b. 安装 Easy Install

> 提示：*需要下载ez_setup.py)

**步骤：** 执行cmd命令 python "D:\software\Jekyll\ez_setup.py"(这里要改为你自己的目录）

**测试：** Easy Install 是否安装成功，执行命令easy_install --version

注意：添加 Python Scripts 路径到环境变量

## c. 安装 Pygments

**步骤：** 执行命令easy_install Pygments

# 5. Jekyll启动与调试

**步骤：** 打开命令行窗口，执行以下命令

```
//创建jekyll工程目录
jekyll new myblog

//切换到工程目录，并开启服务
cd myblog
jekyll serve
```

**测试：** Jekyll是否正常开启，访问[localhost:4000](http://localhost:4000)


# 最后：附件

附件个自动安装脚本，双击install.bat即可：[点此下载](http://or8fqs2b1.bkt.clouddn.com/fastjekyll.rar)