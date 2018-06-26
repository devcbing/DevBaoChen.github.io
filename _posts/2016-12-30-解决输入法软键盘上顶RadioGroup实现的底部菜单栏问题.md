---
layout: post
title: 解决输入法软键盘上顶RadioGroup实现的底部菜单栏问题
categories: [Android, ]
description: 解决输入法软键盘上顶RadioGroup实现的底部菜单栏问题
keywords: Android, 
---

解决输入法软键盘上顶RadioGroup实现的底部菜单栏问题

## 问题如下

本来应该是这样的：

![](http://upload-images.jianshu.io/upload_images/1365793-b6e5e04c06d2c9d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

结果由于在页面中增加了EditText输入框，当打开输入法的软键盘的时候，是下面这样

![](http://upload-images.jianshu.io/upload_images/1365793-d13365ce506dc1e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

纠结一下，这个该如何是好：
## 解决办法
### 方法一：
在这个AndroidManifest.xml中为这个页面的Activity添加如下代码：

```
android:windowSoftInputMode="adjustNothing"
```

![](http://upload-images.jianshu.io/upload_images/1365793-2957b502d08e2dd5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 方法二：
在这个Activity的onCreate()方法里添加下面的代码：
```
this.getWindow().setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_ADJUST_NOTHING);
```

解决：

![](http://upload-images.jianshu.io/upload_images/1365793-9517a528d08c265d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 关于windowSoftInputMode的相关属性

属性 | 说明
---|---
adjustNothing |  （**窗口不做调整**）
adjustPan       |当前窗口的内容将自动移动使用户能总是能看到输入内容的部分（  **软键盘会遮挡屏幕**  ）
adjustResize    |这Activity总是调整屏幕的大小以便留出软键盘的空间（  **可以显示全部屏幕**  ）
adjustUnspecified   |由系统自行决定是隐藏还是显示 （ **默认设置**  ）
stateAlwaysHidden   |就算当前Activty主窗口获得焦点，软件盘也是隐藏的
stateAlwaysVisible    |在当前Activity页面是，软键盘总是显示的状态
stateHidden              |在当前Activity页面是，软键盘总是隐藏的状态
stateUnchanged        |当这个activity出现时，软键盘将一直保持在上一个activity里的状态，无论是隐藏还是显示
sstateUnspecified     |软键盘的状态没有被指定，系统自动选择一个合适的状态或依赖于主题的设置
stateVisible               |软键盘通常是可见的
