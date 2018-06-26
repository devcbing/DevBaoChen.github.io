---
layout: post
title: android studio 打开新的项目 一直Gradle build的问题
categories: [Android, ]
description: 今天新建了一个测试 demo的项目，创建成功后一直在 building 项目，就是打不开，顿时一脸蒙逼 。前几天都可以打开的啊：
keywords: Android, 
---

今天新建了一个测试 demo的项目，创建成功后一直在 building 项目，就是打不开，顿时一脸蒙逼 。前几天都可以打开的啊：

### 问题查找 ：
不管是打开项目或者新建项目，都会出项和 gradle 相关的等待。那么问题应该是和 gradle 相关，好吧，上查找了半天，发现确实是 gradle 的问题，
### 问题解决：
我打开我项目的 `gradle-wrapper.properties` 文件，发现我的 gradle是3.3

```
#Tue Jun 27 16:15:06 CST 2017
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-3.3-all.zip

```
然后查看我的 C:\Users\用户名\.gradle\wrapper\dists 目录下的 gradle-3.3-all 文件中的一个随机生成的文件夹中 的文件，发现只有 


![](http://upload-images.jianshu.io/upload_images/1365793-d6a59958304eb7fb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
但是我的 gradle-2.14.1-all 中有 


![](http://upload-images.jianshu.io/upload_images/1365793-20f694f1b22ad523.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

好吧，现在发现原因了，是因为我没有 

![](http://upload-images.jianshu.io/upload_images/1365793-48305f41b3f75900.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

好的，这时候我们需要去下载相应的 gradle 的压缩包
下载地址：[http://services.gradle.org/distributions/](http://services.gradle.org/distributions/)

下载完后，压缩包复制一份，解压一份，
然后 gradle-3.3-all 中的文件有 

![](http://upload-images.jianshu.io/upload_images/1365793-490d4f27c2205297.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后重新打开刚刚新建的项目， 我勒个去，飞快的打开编译好了，
