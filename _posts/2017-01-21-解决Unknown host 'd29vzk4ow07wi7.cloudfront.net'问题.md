---
layout: post
title: 解决Unknown host 'd29vzk4ow07wi7.cloudfront.net'问题
categories: [Android, Android Studio]
description: 解决Unknown host 'd29vzk4ow07wi7.cloudfront.net'问题
keywords: Android, Android Studio
---

解决Unknown host 'd29vzk4ow07wi7.cloudfront.net'问题

### 问题描述
今天在更新 Android Studio 后，编译运行项目出现下面的问题:

```
Unknown host 'd29vzk4ow07wi7.cloudfront.net'. You may need to adjust the proxy settings in Gradle.
Enable Gradle 'offline mode' and sync project
Learn about configuring HTTP proxies in Gradle
```

### 解决办法：
一般这种情况都是由于网络配置不对引起的 ，所以初步以为是由于 Android Studio 中的 gradle 使用的是 ` offline modle` 但是 取消掉 ` offline modle` 的勾选以后，还是不行，那么还有一种可能是由于 `build.gradle` 中的加载依赖的网络引起的，在 `build.gradle` 关于下载依赖的 配置有 `jcenter()` ;   `mavenCentral()` ; 等。。。，正好发现我的  `build.gradle` 中 没有 `mavenCentral()` ，所以在 jcenter() 前面加上 mavenCentral() , 然后重新编译，一切成功。

