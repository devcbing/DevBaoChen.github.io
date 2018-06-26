---
layout: post
title: ionic3 WKWebView 出现跨域请求的问题
categories: [ionic, ]
description: ionic3 WKWebView 出现跨域请求的问题
keywords: ionic, 
---

### 问题描述：
iOS打包在模拟器中运行，在控制台会看到网络请求后的结果：
```
{"_body":{"isTrusted":true},"status":200,"statusText":"Ok","headers":{},"type":3,"url":null}"

```
### 原因：
因为IONIC3打包iOS时后使用了高性能WebView组建WKWebView WKWebView占用的资源是UIWebView占用的资源的1/40左右，所以IONIC也将默认构建iOS应用的引擎改成了WKWebView。然而 WKWebView 是不支持跨域的，
#### 另外官方对新版IONIC3使用WKWebView的解释

```

```

- 在iOS中，现在有两个网页浏览器，UIWebView和WKWebView。之前的IONIC版本使用的都是UIWebView。现在都将使用WKWebView。
- 我们坚信WKWebview是任何应用程序的最佳选择，因为它比旧版的webview（UIWebView）有许多改进。这些功能包括：
- 将JS代码JIT转换为机器代码，运行速度更快
    1. 改进的渲染性能
    2. 更少的内存消耗
    3. 更好地遵守网络标准
    4. 可靠的滚动事件（对虚拟列表很重要）

### 解决办法
######  一、后端配置，允许接口跨域
###### 二、强制cordova使用UIWebView引擎渲染页面
(1)、在config.xml里增加以下配置
```
<preference name="CordovaWebViewEngine" value="CDVUIWebViewEngine" />
```
(2)、卸载WKWebView插件
```
$ ionic cordova plugin remove cordova cordova-plugin-ionic-webview --save
$ rm -rf platforms/
$ rm -rf plugins/
$ ionic cordova build ios
```

参考：
[https://ionicframework.com/docs/wkwebview/](https://ionicframework.com/docs/wkwebview/)
