---
layout: post
title: android studio生成签名导打包的方法
categories: [Android, Android Studio]
description: 使用 Android Studio 自动生成签名并打包
keywords: Android, Android Studio
---

## 方法一：
在android中。可以非常快速的生成签名文件.jsk文件。步骤如下：
### 第一步：

![](http://upload-images.jianshu.io/upload_images/1365793-2ba848ad2c428608.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 第二步：

![](http://upload-images.jianshu.io/upload_images/1365793-3915850376d315a6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如果你已经有了签名文件.jsk那么就选择③导入文件，这时①中就是文件路径，④是keystore的密码，⑤是别名，⑥是文件的密码。

我们这里默认没有.jsk文件。所以点击②新建一个.jsk文件

### 第三步：

![](http://upload-images.jianshu.io/upload_images/1365793-167d7c884a8bcdc3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###  第四步：
这里我填写的全是模拟的数据：

![](http://upload-images.jianshu.io/upload_images/1365793-a43f7cc42e1e8d87.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![](http://upload-images.jianshu.io/upload_images/1365793-a7d9f0369e4198c2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



![](http://upload-images.jianshu.io/upload_images/1365793-72e9c02d84a3163a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

路径可选择：我这选择的是：F:\daima\TestJsk\app
点击finish：

![](http://upload-images.jianshu.io/upload_images/1365793-7926465f04488092.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到生成一个：app-release.apk
同时生成一个：testjsk.jks



![](http://upload-images.jianshu.io/upload_images/1365793-4e98ddbf552ce7b8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



ok:完成


## 方法二：

### 第一步：

这种方法是在假设你已经有了jsk文件的前提下面(这里我们用在第一中方法中生成的testjsk.jks)
![](http://upload-images.jianshu.io/upload_images/1365793-084443fe666413bf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到在app的build.gradle中生成了:
```
signingConfigs {
        config {
            keyAlias 'test'
            keyPassword '123456'
            storeFile file('F:/daima/TestJsk/testjsk.jks')
            storePassword '123456'
        }
    }
```
### 第二步：

![](http://upload-images.jianshu.io/upload_images/1365793-4a5284071bf26966.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这里的Singing Config选择在  前一步的config即可；

可以看到在app的build.gradle中生成了:
```
buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.config
        }
        debug {
            signingConfig signingConfigs.config
        }
    }
```
在android studio中的terminal 中使用gradlew assembleRelease命令，可以在outputs的apk中生成签名后的apk文件
![](http://upload-images.jianshu.io/upload_images/1365793-3f85e57bfcb15ab8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 如何查看签名后的jsk中的信息

找到java的jre的bin下的keytool.exe
在cmd中输入下面命令：keytool -list -v -keystore "jsk路径" -storepass 密码

![](http://upload-images.jianshu.io/upload_images/1365793-c40a106028e13d01.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


需要注意的是签名密码千万不要暴露：
