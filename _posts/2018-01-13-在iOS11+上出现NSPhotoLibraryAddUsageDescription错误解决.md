---
layout: post
title: 在iOS11+上出现NSPhotoLibraryAddUsageDescription错误解决
categories: [XCode, iOS]
description: 在iOS11+上出现NSPhotoLibraryAddUsageDescription错误解决
keywords: XCode, iOS
---

在iOS11+上出现NSPhotoLibraryAddUsageDescription错误解决

### 问题描述:
在 手机做头像上传功能时，出现下面的错误：
```

This app has crashed because it attempted to access privacy-sensitive data without a usage description.  The app's Info.plist must contain an NSPhotoLibraryAddUsageDescription key with a string value explaining to the user how the app uses this data.
(lldb) 
```


### 问题原因:
由于手机刚刚好升级 iOS11.4的系统，从错误描述来看，是权限问题，只需要在 .plist 中加入 `NSPhotoLibraryAddUsageDescription` 相关权限；

### 解决办法：
在 .plist 中加入 `Privacy - Photo Library Additions Usage Description` ，并做好描述.

 ![](/images/posts/tools/2018-01-13-01.png)

参考文档：
[https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html)

