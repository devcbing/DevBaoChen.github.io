---
layout: post
title: 在同一台android设备上安装多个同一项目的apk
categories: [Android, Android Studio]
description: Android 使用 Android Studio 进行多渠道打包
keywords: Android, Android Studio
---
### 前言
通常我们在一台android设备上(无论是真机还是模拟器)，安装相同包名的apk应用都只能安装一个应用，如果设备上已经安装了一个apk，如何再次安装这个apk就会覆盖前面的应用，如果想要安装在同一台设备上安装多个相同的apk，唯一的办法只能是改包名了。简单的项目还好，当个一个项目中有许多的类的时候，手动去更改包名一定是一种不可取的方式。那么有没有什么简单的办解决呢？
### 解决办法：
在app下面的`bulid.gradle`中添加如下代码：
```
 buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
        debug {
            applicationIdSuffix ".free"
        }
    }
}
```
没错，你可以看到`applicationIdSuffix ".free"`这一行关键的代码。就是这一句代码，使得我们的包名变成了`packageName.free`,比如：我们包名是`com.myApp`,加上这句后，生成的apk的包名就会变成`com.myApp.free`,这里的`free`可以根据自己的打包用途来命名。
上这种方式就是`BuildTypes`,可以在下面的图中看到：








![BuildTypes方式](http://upload-images.jianshu.io/upload_images/1365793-258486b8b820c712.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 另外一种方式Flavors
```
    productFlavors {
        pro {
            applicationIdSuffix ".pro"
        }
        free {
            applicationIdSuffix ".free"
        }
    }
```
或者：
```
    productFlavors {
        pro {
            applicationId = "com.myApp.pro"
        }
        free {
            applicationId = "com.myApp.free"
        }
    }
```

![Flavors方式](http://upload-images.jianshu.io/upload_images/1365793-31657bc79ba04846.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这时候使用Build Variants就可以选择自己需要打包的版本

![](http://upload-images.jianshu.io/upload_images/1365793-2bc1083fdb1a43fe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
下面是我生成的apk
![](http://upload-images.jianshu.io/upload_images/1365793-78a6fb1382ff178e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

手机上安装的应用：

![](http://upload-images.jianshu.io/upload_images/1365793-c04bae319e8a6fae.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
上面三个应用都是同一个项目，只是经过处理后，就都可以在同一台设备上安装了(因为这是三个的包名是不相同的)。
需要注意的是：
```
     productFlavors {
        pro {
            applicationId = "com.myApp.pro"
        }
        free {
            applicationId = "com.myApp.free"
        }
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
        debug {
            applicationIdSuffix ".test"
        }
    }
```
如果同时使用两种方式，buildTypes 中的设置后缀会跟在productFlavors 中设置的包名后面,这时候包名就会变成`com.myApp.pro.test`和`com.myApp.free.test`.
Ok这时我们就可以在同一台设备上安装多个同一项目的apk了
### 总结：
`Application ID`可以说是一个android项目的唯一标识，是不可修改的，在android studio中的bulid.gradle下，如下：
```
android {
    compileSdkVersion 25
    buildToolsVersion "25.0.2"
    defaultConfig {
        applicationId "com.myApp"
        minSdkVersion 15
        targetSdkVersion 25
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
```
并且Application ID和package的属性值不一样的，虽然不一样，但是当我们在构建项目的时候，会复制Application ID给package的属性值,并且这个属性值是唯一的，package属性才是真正作为您应用程序唯一身份凭证。

当我们打开apk中的AndroidMainifest.xml文件时就可以看到，package和我们设置的一样了。
这就是为什么可以在同一台设备上安装的原因。
如何我们的apk已发布，这个项目的包名就是不可修改的了，不过我们一般也不会轻易修改包名。

参考:[https://developer.android.com/studio/build/application-id.html#change_the_package_name](https://developer.android.com/studio/build/application-id.html#change_the_package_name)
