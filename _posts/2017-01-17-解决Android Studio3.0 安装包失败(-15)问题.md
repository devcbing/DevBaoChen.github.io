---
layout: post
title: 解决Android Studio3.0 安装包失败(-15)问题
categories: [Android, Android Studio]
description: 解决Android Studio3.0 安装包失败(-15)问题
keywords: Android, Android Studio
---

解决Android Studio3.0 安装包失败(-15)问题

### 问题描述：
前面几天把 Android Studio  升级到3.0以后，就自己进行开发了，用 USB 在自己的手机上调试， 比在模拟器上好多了，美滋滋。突然，同时让我给他发一个安装包 ，发就发，不怕 ，没过1分钟。同事说他的手机安装不上 apk  ，what??? ,我又在我手机上运行了一次，可以的啊。 发生了什么，又去借了其他同事的手机来测试，发现他们都安装不上，完了 ，这下问题大了。然后去上上一个版本的 apk 发现一切正常， what ？？？。我什么都没有做啊，就升级了一下 Android Studio ，等等，升级 Android Studio ，难道又是 Android Studio3.0  的坑，
没办法了，只好 google 一下，发现还是有很多人和我遇到同样的问题。

### 解决办法：
由于我都是用的 USB 在我电脑上运行，这种使用 adb 方式得到的 apk 包，发送出去后，就不能安装了， 会出现 （-15）错误。但是使用 `build apk `命令，或者 下图这样的 gradle 命令打包的 apk 就不会出现问题. 

![](/images/posts/android/2018-01-17-01.png)

可能这也是 Android Studio 的一个坑吧。

