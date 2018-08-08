---
layout: post
title: Command /usr/bin/codesign failed with exit code 1错误的解决办法
categories: [XCode, ]
description: some word here
keywords: XCode, 
---

Command /usr/bin/codesign failed with exit code 1错误的解决办法

### 问题描述

在使用 XCode 在真机上打包运行iOS 项目的时候出现 `Command /usr/bin/codesign failed with exit code 1` 错误，但是前面几分钟 打包的时候都是正常的，一脸懵逼，

### 寻找问题解决办法

1. 怀疑是项目中出现错误，去排查，未发现错误
2. 怀疑是 XCode 的问题， 重启 XCode ,问题未能解决
3. 上  Google ,未能知道具体导致错误原因， 但是提供一个解决方案 `重启电脑`

### 解决办法
` 重启电脑`：重启电脑， 重新打开 XCode , 打包编译， 运行成功

### 总结

Mac OS 抽风？？？

