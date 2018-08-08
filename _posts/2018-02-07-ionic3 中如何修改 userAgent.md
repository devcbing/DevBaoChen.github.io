---
layout: post
title: ionic3 中如何修改 userAgent
categories: [ionic, ]
description: ionic3 中如何修改 userAgent
keywords: ionic, 
---

ionic3 中如何修改 userAgent

### 问题描述
我们知道，在浏览器中 都会有 `userAgent` 比如 
```
navigator.userAgent
"Mozilla/5.0 (iPhone; CPU iPhone OS 11_0 like Mac OS X) AppleWebKit/604.1.38 (KHTML, like Gecko) Version/11.0 Mobile/15A372 Safari/604.1"
```
同样的， 在 ionic 的项目中也会存在 `userAgent` ，那么我们如何修改他呢

### 解决办法：

在 ionic 生成的 app 中，我们其实使用的是  Cordova 来打包成一个 `CordovaWebView` ,那么，如果我们想修改  `userAgent` ,就需要修改 Cordova 的相关配置， 在 ionic 项目中，Cordova 是在 `Config.xml` 中 , 我们查看 [文档](https://cordova.apache.org/docs/en/8.x/config_ref/index.html)， 知道， 修改  `userAgent` 有下面两种方法：

Attributes(type)  | Description
---|---
AppendUserAgent(string)    |   If set, the value will append to the end of old UserAgent of webview. When using with OverrideUserAgent, this value will be ignored.
OverrideUserAgent(string)  | If set, the value will replace the old UserAgent of webview. It is helpful to identify the request from app/browser when requesting remote pages. Use with caution, this may causes compitiable issue with web servers. For most cases, use AppendUserAgent instead.

#### 第一种方法：

`AppendUserAgent(string) ` ，会在原有的  `userAgent`  末尾添加新内容 ;
使用方式是在 `Config.xml` 中 添加下面的代码;

```
<preference name="AppendUserAgent" value="My Browser" />
```
现在的 `userAgent` 是 `Mozilla/5.0 (iPhone; CPU iPhone OS 11_0 like Mac OS X) AppleWebKit/604.1.38 (KHTML, like Gecko) Version/11.0 Mobile/15A372 Safari/604.1 My Browser`

#### 第二种方法：

`OverrideUserAgent(string)` 是直接覆盖掉 原来的 `userAgent` 

使用方式是在 `Config.xml` 中 添加下面的代码;

```
<preference name="OverrideUserAgent" value="Mozilla/5.0 My Browser" />
```
现在的 `userAgent` 是 `Mozilla/5.0 My Browser`;

### 总结：

我们在 ionic 中 修改其 `userAgent` 的 时候最好使用  `AppendUserAgent(string) ` 方法，这个样不会替换掉 原有的 `userAgent` 。否则 如何原有的 `userAgent` 被替换了， 我们很可能 不能使用 `import { Platform,} from 'ionic-angular';` 来判断是什么平台。

参考： 
- 1. [https://cordova.apache.org/docs/en/8.x/config_ref/index.html](https://cordova.apache.org/docs/en/8.x/config_ref/index.html)
- 2. [https://stackoverflow.com/questions/32052221/how-i-can-set-user-agent-in-cordova-app/32337353#32337353](https://stackoverflow.com/questions/32052221/how-i-can-set-user-agent-in-cordova-app/32337353#32337353)
