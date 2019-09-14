---
layout:     post
title:      "Web文件上传实现"
subtitle:   "Nodejs接收"
date:       2018-09-10
author:     "BoomXu"
tags:
    - JavaScript
    - Node.js
---

> 项目匿名评论系统的文件上传服务的实现

做了个匿名评论论坛的小系统，里面涉及到了图片的上传，期间遇到的问题汇总。

## HTML

首先上传文件还是要套到form表单里，并且文件上传是 input type = file 的一个标签‘

```
<form id="form1" method="POST" enctype ="multipart/form-data">
  <div class="form-group">
    <label class="color" for="exampleInputFile">图片发表</label>
    <input class="color" type="file"  id="exampleInputFile" multiple="multiple" onchange="imagechange(this)" value="">
  </div>
  <textarea style="resize:none;" name="content" class="form-control" rows="9"></textarea>
  <button style="margin-top:10px;" type="button" onclick="upload1()" class="btn btn-primary btn-lg btn-block">发表发表发发表发表发表表发发表发表</button>
</form>
```

这里上表单上传多了一个 `enctype ="multipart/form-data"` 属性，这是代表表单可以上传带有数据的文件。input type = file 的控件是文件标签，其中 `class="form-control"` 是可以上传多个文件？ （但我实践上这个属性并没有什么用，最后还是在js里push了多文件上传） `onchange="imagechange(this)"` 这个是在文件选择被改变的事件，这里把本dom对象传给js就不用麻烦的获取该dom对象了。

其中该dom节点有一个files对象，这是一个数组，里面放了文件的对象，按道理来说 `enctype ="multipart/form-data"`会在这里生效在多文件的时候往files里添加多个文件，但实际并没有，新的文件还是把旧的文件覆盖了。所以下面我手动push到了一个新的数组里。**这地方还有待研究**

注意的是这里一个坑，**当我把函数名命名为 upload 的时候，居然不会生效？** 难道是函数命名有冲突？

## JS
```
function imagechange(a) {
  files.push(a.files);
}
function upload1() {
  var fd = new FormData(document.getElementById('form1'))
  files.forEach(item => {
    fd.append('pic',item[0])
  });
  fd.append('postuser','小兵旭旭')
  $.ajax({
    url: "/sendpost",
    type: "POST",
    data: fd,
    sync: false,
    processData: false,  // 告诉jQuery不要去处理发送的数据
    contentType: false,   // 告诉jQuery不要去设置Content-Type请求头
    success: function(response,status,xhr){
      console.log(xhr);
      if (response === 'success') {
        $('#sendpost').toggle();
        location.reload()
      }
    }
  });
}
```

js里是用了新的 FormData 对象，这个对象可以接收一个form表单的对象，并且 FormData **可以自己添加属性的**，他会一并跟随上传到服务器上。这里把文件数组添加了进去，再通过ajax异步发送到服务器，摆脱了页面停止响应的异步表单提交。

不过需要注意的是，直接console.log（FormData）的对象看上去是个空的对象，**他传的参数是不直接在对象里的，而在对象的更深处** 这种情况可以在 network 里查看到传出的参数

## Node...

在Node接收文件的时候也有许多坑，其中就是 express 接收post的请求，**并且含有文件数据请求的时候，需要一个中间件**

```
const multiparty = require('connect-multiparty')
.use(multiparty({uploadDir:'./linshi'}))
var mutipartMiddeware = multiparty();
.post('/sendpost', mutipartMiddeware, (req, res) => {})
```

{uploadDir:'./linshi'} 配置了该中间件接收到的文件存放的临时目录。并且该中间件接收到的res有以下几个有用的参数：

```
req.body: 请求体
req.files: 接收到的文件数组
req.files.file.path: 临时存放文件的路径
req.files.file.name: 接收到的文件名字
```

上面注意到，该中间件存放的是临时文件，并且文件名是随机命名的（req属性里有真实的文件名字），如果想正常保存上传来的文件，还是需要自己手动去处理：

```
var target_path = './upload/' + req.files.pic.name
fs.rename('./' + req.files.pic.path, target_path, function (err) {
  if (err) throw err;
  postObj.images.push(target_path.replace('.',''))
  // 删除临时文件夹文件, 
  fs.unlink('./' + req.files.pic.path, function () {
    if (err) throw err;
  });
```

这样一个基本的文件上传服务就完成了，当然还有很多判断需要自己手动去实现。这里是使用了 FormData 对象，MDN上也给出了不使用该对象的方法。

[FromData上传文件的方法](https://developer.mozilla.org/zh-CN/docs/Web/API/FormData/Using_FormData_Objects)

[Ajax序列化及文件上传](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/Using_XMLHttpRequest#Submitting_forms_and_uploading_files)