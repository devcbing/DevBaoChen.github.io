---
layout: post
title: Android Studio 打包出现:app:lint FAILED错误
categories: [Android, Android Studio]
description: Android Studio 打包出现:app:lint FAILED错误
keywords: Android, Android Studio
---

Android Studio 打包出现 :app:lint FAILED 错误

### 问题描述：

在 AS 中执行 `assemble` 命令打包的时候出现 `app:lint FAILED` 错误；

### 解决办法：

在 app 的 build.gradle 中增加 android 下面增加下列代码：

```
    lintOptions {
        checkReleaseBuilds false
        abortOnError false
    }
```

### 原因分析
 
在代码中使用一些过时的 API , 当在使用 `assemble` 命令进行打包的时候，会去检查代码，上面的解决办法，是忽略代码被 lint 检查。


