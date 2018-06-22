---
layout: post
title: 无需root查看手机中数据库和SharedPreferences 中的数据
categories: [Android, ]
description: 无需root查看手机中数据库和SharedPreferences 中的数据
keywords: Android, 
---

无需root查看手机中数据库和SharedPreferences 中的数据

### 方式一：
`stetho`没错，就是stetho，这个facebook出品的一款在线调试的东西;
stetho的官网:[http://facebook.github.io/stetho/](http://facebook.github.io/stetho/)
使用stetho方式：
####  第一步：
在android studio中的app的build.gradle中引入：
```
compile 'com.facebook.stetho:stetho:1.4.2'
```
####  第二步：
在项目的Application中进行初始化：
```
 @Override
    public void onCreate() {
        super.onCreate();
        Stetho.initializeWithDefaults(this);
    }
```
####  第三步：
运行你的程序:这时候需要注意的是：
保证你的手机可以通过usb链接电脑，并且是在开发者模式的debug模式；
然后打开`chrome浏览器`(google浏览器)；
在浏览器的地址栏中输入下面的地址：
`chrome://inspect/#devices`
这时候会看到下面的界面：


![调试界面一](http://upload-images.jianshu.io/upload_images/1365793-98c256b74f32cd50.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
然后点击你的项目包名下的`inspect`


![选中inspect](http://upload-images.jianshu.io/upload_images/1365793-14a84eec9c2ff092.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后就会跳到真正的调试界面了:

![调试界面](http://upload-images.jianshu.io/upload_images/1365793-679bbcb69d7bd0af.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

ps:如果打不开界面；这时候需要**`科学上网`**
我们可以看到，这个和web前端的调试界面相差无几：
现在我们就可以在`resource`中看到看到我们手机上运行的程序的相关数据库和SharedPreferences 等等数据了:

![CLALCDKFLU%U3P491{FFU1P.png](http://upload-images.jianshu.io/upload_images/1365793-e1eda736ba651161.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

细心的你可能会发现，我们不仅能够查看数据库中额数据。还可以查看我们的布局等等相关信息；就和前端调试一样，爽的不要不要的；

![](http://upload-images.jianshu.io/upload_images/1365793-021b432aea023e15.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果需要调试网络，我们可以在stetho的官网[http://facebook.github.io/stetho/](http://facebook.github.io/stetho/)看到，只需要在build.gradle配置相关信息即可：

![](http://upload-images.jianshu.io/upload_images/1365793-eb0ffda0eb5db6cf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当然：stetho的调试功能不仅是这是，还有其他功能需要自己去试试才知道。

上面stetho的方法必须要使得手机等设备链接电脑，如果没有带数据线怎能办？？？
下面简单的介绍一下最近发现的通过`wifi`来进行查看数据库数据/SharedPreferences 文件中数据等信息的方式；
### 方式二：

先上给github地址：[https://github.com/amitshekhariitbhu/Android-Debug-Database](https://github.com/amitshekhariitbhu/Android-Debug-Database)

没错，作为菜鸟的我需要站在大神的肩上：
`Android-Debug-Database`的使用方法：
#### 第一步：
同样是在build.gradle中添加：
```
   debugCompile 'com.amitshekhar.android:debug-db:1.0.0'
```
debugCompile 只是为了在debug模式下调试使用，如果是正式版，为了数据，安全，建议去掉；
然后运行：
现在你会问了？就这样：WTF，我该怎么门槛我的数据库中的相关数据？

不要急。下面来看第二步：
####  第二步
打开你的浏览器：输入地址。这时候，你肯又要问了，输入什么地址，我没看见有地址啊：
如果你不知道输入什么地址，这时候你就需要在你的启动页的activity中加入下面的代码了：
```
DebugDB.getAddressLog();
```
额，其实在其他地方也可以，比如：Application中，或者其他任意地方，反正这行代码只是用来打印前面说到的地址的：
这是后运行程序，我们就可以在log日志中看到地址：



![LOG中的地址](http://upload-images.jianshu.io/upload_images/1365793-f14094eeb4817292.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在浏览器中输入地址就可以看到：

![数据展示](http://upload-images.jianshu.io/upload_images/1365793-14ff0b62b260bb72.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如果你需要修改8080端口只需要在build.gradle中的buildTypes下面你的debug中进行配置：
```
  buildTypes {
      debug {
            resValue("string", "PORT_NUMBER", "8081")
        }
}
```

现在地址变成了`http://192.168.1.106:8081 `
![](http://upload-images.jianshu.io/upload_images/1365793-971fb3a9f2344a6b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/1365793-693c1a71ce002948.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这种方式的要求就是：`保证手机等设备和电脑在同一wifi下面`。


###  总结
两种无需root查看手机中数据库和SharedPreferences 中的数据的方式各有优缺点，第一种，功能十分强大，但是需要使用usb链接设备和电脑。
在只是查看数据，不调试的情况下，第二方式要比第一种方式方便许多，(不过一般运行在手机上的时候都会使用数据线链接，所以差别不大)。
关于两种方式的其他技巧，需要去阅读相应的官方介绍。

