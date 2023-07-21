---
title: ReactNative踩坑记
date: 2020-07-21 12:16:24
tags:
  - ReactNative
---

## 最近在使用react-native的时候遇到了很多坑,这里记录一下

## 样式
react-native 虽然支持flex布局，但是所有的样式均是css样式的一个很小的集合，尤其是在安卓机下问题尤为凸显：

1. View内部的元素千万不要超出父级的范围，iso上问题倒是不大，安卓上就什么超出的都看不到了

2. lineHeight 可以用，不过千万不要写成小数，否则安卓上会直接崩溃

3. rn的样式不存在继承的情况，所以基本上每个节点都要写style，真的是体力活

4. 如果Text的父级元素设置了背景颜色，那么ios下Text的背景颜色也是父级的背景颜色，要么自己写个Text重置下样式，要么就遇到了再改

5. react-native的字号是没有设置单位的，所以会随着系统设置的字体大小而变化，我也不知道这是不是坑，不过貌似有的app也没有管这个，如果硬要去设置Text的文字不随系统改变，安卓是可以统一设置的，ios上Text设置allowFontScaling ={false}就可以解决

## 异常
react-native 在发生js异常的时候，debug的时候会直接红屏幕，但是再release的时候直接会崩溃退出，解决办法

```JavaScript
import ErrorUtils from "ErrorUtils" <br>//这里应该做个判断，如果不是debug的才做这样的异常全局处理
ErrorUtils.setGlobalHandler((e)=>{<br>　　//发生异常的处理方法,当然如果是打包好的话可能你找都找不到是哪段代码出问题了
　　Alert.alert("异常",JSON.stringify(e))
});
```

## fetch
react-native虽然自带有fetch，不过在使用的时候发现了一个问题，如果需要获取http的header头的时候问题就来了，可能得到的是一些千奇百怪的样式，这并不是react-native的错，而是第三方的 whatwg-fetch 留下的坑，当然也有人再github上跟react-native反映过这个问题，不过得到的解决方案都很坑，唯有一个办法，就是拷贝自己修改，修改如下:

1. 注释该注释的
```JavaScript
(function(self) {
    'use strict';
　　 //注释这里，不然总是用的是全局的fetch
    // if (self.fetch) {
    //     return
    // }
```

2. 修改该修改的
```js
function parseHeaders(rawHeaders) {
    var headers = new Headers()<br>　　　　　//把\t\n改成\t，因为一般header都是用\n来分割的
    rawHeaders.split('\n').forEach(function(line) {
    //rawHeaders.split('\t\n').forEach(function(line) {
            var parts = line.split(':')
            var key = parts.shift().trim()
            if (key) {
                var value = parts.join(':').trim()
                headers.append(key, value)
            }
        })
    }
    return headers
}
```
3. 直接import你改好的文件，fetch就可以用了

## Modal
Mode控件在使用的时候要注意了，因为这个是rn提供的，并且也写的很清楚是最高层级的一个弹出层，所以你想要又打开Model又要跳转基本是无望的了，所以建议不要使用这个，最好是使用第三方的控件，我们用的是 react-native-modalbox + 高阶控件 实现的全遮盖的弹出层

## 点击屏幕其他位置关闭的菜单
这类菜单有个共同的特点就是点击屏幕其他地方然后菜单就关闭，我们的解决办法就是用自己写的 react-native-modalbox + 高阶控件 也就是说放在一个弹出层里面，当然可以试试把当前页面套进一个大的 TouchableWithoutFeedback 里面

## 接口请求
非特殊情况下都应该这样做

```js
import {InteractionManager} from "react-native"
componentDidMount(){
  InteractionManager.runAfterInteractions(() => {
      fetch("xxx.xxx.xxx",{})
    });     
}
```

## 键盘
官方提供的自定义隐藏键盘的方法是

```js
import { Keyboard } from 'react-native'
Keyboard.dismiss()
```
但是我试了很多次之后发现根本不能，而且还报错，楼主的react-native版本是0.35.0

看了官方的issue才知道这个不行，推荐下面方法
```js
import dismissKeyboard from 'dismissKeyboard'dismissKeyboard()
```
这样就可以隐藏了，太坑了

还有个很坑的地方，官方提供的移除键盘事件的方法不可用

```js
componentDidMount () {　　Keyboard.addListener('keyboardDidShow', this.keyboardDidShow.bind(this))　　Keyboard.addListener('keyboardDidHide', this.keyboardDidHide.bind(this))}componentWillUnmount () {
    Keyboard.removeAllListeners('keyboardDidShow')
    Keyboard.removeAllListeners('keyboardDidHide')
}
```

这样的方式特么的如果操作快了，或者有时候莫名其妙的就会出错,下面的才是正确的打开方式：
```js
componentDidMount () {
　　this.keyboardDidShowListener = Keyboard.addListener('keyboardDidShow', this.keyboardDidShow.bind(this))
　　this.keyboardDidHideListener = Keyboard.addListener('keyboardDidHide', this.keyboardDidHide.bind(this))
}
componentWillUnmount () {
    this.keyboardDidShowListener.remove()
    this.keyboardDidHideListener.remove()
}
```

## https
https这个问题上ios还好，安卓问题就来了，前期我们准备将ajax请求的库丢给原生安卓和ios来做我们直接调用就是了，但是后来发现问题这样那样的问题太多了，

所以在热更新服务器启动或者打包的时候就把源代码先改了在进行打包或者启动服务器

文件位置：
```
node_modules/react-native/ReactAndroid/src/main/java/com/facebook/react/modules/network/OkHttpClientProvider.java
```
这个文件的最后一个方法修改如下：
```js
private static OkHttpClient createClient() {
  // No timeouts by default
  return new OkHttpClient.Builder()
  .sslSocketFactory(sslContext.getSocketFactory())
  .hostnameVerifier(new HostnameVerifier() {
      @Override
      public boolean verify(String hostname, SSLSession session) {
              return true; //忽略所有的认证，直接返回了true
          }
        })
    .connectTimeout(0, TimeUnit.MILLISECONDS)
    .readTimeout(0, TimeUnit.MILLISECONDS)
    .writeTimeout(0, TimeUnit.MILLISECONDS)
    .cookieJar(new ReactCookieJarContainer())
    .build();
}
```
修改源代码的方式有点略坑，不过可以解决很多问题，还节约时间！！！

## BackAndroid
安卓机有独特的点击按键返回，所以在最外层会注册一个监听方法

```js
bindHardwareBackPress(){
    if (Platform.OS === 'android') {
        BackAndroid.addEventListener('hardwareBackPress', this._onHomeBackPress);
    }
}
 
onHomeBackPress(){
    let routeList = this.getRouteList();
    if (routeList.length !== 1) {
        this.navigator.pop();
        return true;
    }
 
    this.handleHomeBackPress();
    return true;
}
 
handleHomeBackPress(){
    if (Platform.OS === "android") {
        ToastAndroid.show("再按一次退出应用", ToastAndroid.SHORT);
        BackAndroid.removeEventListener("hardwareBackPress", this._onHomeBackPress);
        BackAndroid.addEventListener("hardwareBackPress", this._onExitApp);
        this.timer = TimerMixin.setInterval(() => {
            TimerMixin.clearInterval(this.timer);
            BackAndroid.removeEventListener("hardwareBackPress", this._onExitApp);
            BackAndroid.addEventListener("hardwareBackPress", this._onHomeBackPress);
        }, 2000);
    }
}
 
exitApp(){
    BackAndroid.exitApp();
}
```

上面的代码是监听返回键，如果不是在最外层的路由就返回上一个,如果在最外层就直接关闭app，但是有很多这样那样的需求要去对安卓的返回键进行操作，坑就来了，你以为提供的removeEventListener方法是没问题的？no ！！！ 他会移除所有的监听，这是不是很坑！！！！

所以：**在需要对安卓返回键进行特殊处理的时候记得其他地方做了监听的再重新监听一次！！！！**