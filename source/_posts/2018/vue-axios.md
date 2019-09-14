---
layout:     post
title:      "Vue-resouce改为Axios之旅"
subtitle:   "Axios"
date:       2018-08-28
author:     "BoomXu"
tags:
    - Vue
---

> 尤雨溪说了，vue2.0 将不再维护自己的原生请求插件 vue-resource 而推荐使用axios，所以以后就用axios来发送vue请求了。

## 使用 cnpm 安装 axios

`cnpm install axios --save-dev`

安装其他插件的时候，可以直接在 main.js 中引入并 Vue.use()，但是 axios 并不能 use，只能每个需要发送请求的组件中即时引入
为了解决这个问题，有两种开发思路，一是在引入 axios 之后，修改原型链，二是结合 Vuex，封装一个 aciton。这里只说修改原型链的方式
改写原型链

## 首先在 main.js 中引入 axios

`import axios from 'axios'`

这时候如果在其它的组件中，是无法使用 axios 命令的。所以我们将 axios 改写为 Vue 的原型属性

`Vue.prototype.$http= axios`

在 main.js 中添加了这两行代码之后，就能直接在组件的 methods 中使用 $http命令
例如

```
methods: {
  show() {
    this.$http({
      method: 'get',
      url: '/user',
      data: {
        name: 'virus'
      }
    })
}
```

## 配置 axios

实际上只有 url 是必须的，完整的 api 可以参考
[https://link.jianshu.com/?t=https://www.kancloud.cn/yunye/axios/234845](https://link.jianshu.com/?t=https://www.kancloud.cn/yunye/axios/234845)

1.  对于get请求

```
axios.get('/user', {
  params:{
        name:"virus"  
  }
})
```

2.  对于POST请求

```
axios.post('/user',{
  name:"virus" 
})
```

3.  一次并发多个请求

```
function getUserAccount(){
  return axios.get('/user/12345');
}
function getUserPermissions(){
  return axios.get('/user/12345/permissions');
}
axios.all([getUserAccount(),getUserPermissions()])
  .then(axios.spread(function(acct,perms){
    //当这两个请求都完成的时候会触发这个函数，两个参数分别代表返回的结果
  }))
```

4.  axios可以通过配置（config）来发送请求

```
axios({
  method:"POST",
  url:'/user/1111',
  data:{
    name:"virus" 
}
});
```

**完整的请求还应当包括 .then 和 .catch**

```
.then(function(res){
    console.log(res)
  })
.catch(function(err){
    console.log(err)
  })
```

当请求成功时，会执行 .then，否则执行 .catch
这两个回调函数都有各自独立的作用域，如果直接在里面访问 this，无法访问到 Vue 实例,这时只要添加一个 .bind(this) 就能解决这个问题

```
.then(function(res){
    console.log(this.data)
}.bind(this))
```

## 请求方式的别名，这里对所有已经支持的请求方式都提供了方便的别名

```
axios.request(config);

axios.get(url[,config]);

axios.delete(url[,config]);

axios.head(url[,config]);

axios.post(url[,data[,config]]);

axios.put(url[,data[,config]])

axios.patch(url[,data[,config]])

```