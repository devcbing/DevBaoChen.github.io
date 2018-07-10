---
layout: post
title: android studio界面简单介绍
categories: [Android, Android Studio]
description: android studio界面简单介绍
keywords: Android, Android Studio
---

## 首先简单介绍项目的结构：
项目结构如下图：

<!-- ![](http://upload-images.jianshu.io/upload_images/1365793-7730f3ace0facce3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

具体信息：
```
 idea：//AS生成的工程配置文件，类似Eclipse的project.properties。
 app：//AS创建工程中的一个Module。
 gradle：//构建工具系统的jar和wrapper等，jar告诉了AS如何与系统安装的gradle构建联系。
   build：//构建目录，相当于Eclipse中默认Java工程的bin目录
        编译生成的apk也在这个目录的outs子目录，不过在AS的工程里是默认不显示out目录的，就算有编译结果也不显示，右键打开通过文件夹直接可以看。
    libs：//依赖包，包含jar包和jni等包。
    src：//源码，相当于eclipse的工程。
    main：//主文件夹 
        java：//Java代码，包含工程和新建是默认产生的Test工程源码。 
        res：//资源文件，类似Eclipse。
            layout：//App布局及界面元素配置，雷同Eclipse。
            menu：//App菜单配置，雷同Eclipse。 
            values：//雷同Eclipse。
                dimens.xml：//定义css的配置文件。 
                strings.xml：//定义字符串的配置文件。 
                styles.xml：//定义style的配置文件。
                ......：//arrays等其他文件。
            ......：//assets等目录
        AndroidManifest.xml：//App基本信息（Android管理文件） 
        ic_launcher-web.png：//App图标 
    build.gradle：//Module的Gradle构建脚本
```
## 然后介绍android studio的主界面

![](http://upload-images.jianshu.io/upload_images/1365793-20f9334b681c9a9a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## android studio和eclipse的比较
具体说就是：

    android studio是单工程的开发模式
    android studio中的application相当于eclipse里的workspace概念
    android studio中的module相当于eclipse里的project概念

## android studio的工程根目录下的build.gradle文件：

![](http://upload-images.jianshu.io/upload_images/1365793-f2783711138291ce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## android studio的工程根目录下的Module的build.gradle文件

![](http://upload-images.jianshu.io/upload_images/1365793-8fe9ca3ffad3a084.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## android studio中的其他常用设置：
### 1.设置主题：
![](http://upload-images.jianshu.io/upload_images/1365793-d71b883343573ac7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 2.设置快捷键

![0{MXAJJQZ9{TSHG0_M1VPCK.png](http://upload-images.jianshu.io/upload_images/1365793-c3b99d12edb91469.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 3.自定义代码属性：

![](http://upload-images.jianshu.io/upload_images/1365793-4d94006073dfd9a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 4.版本控制：

![设置中的版本控制](http://upload-images.jianshu.io/upload_images/1365793-02813c1650afa9e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![工具栏中的版本控制](http://upload-images.jianshu.io/upload_images/1365793-261f78c9585b9d90.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 5.选择/设置模拟器

![](http://upload-images.jianshu.io/upload_images/1365793-e643755f483f16d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 6.还有其他，啦啦啦啦~~~ -->











