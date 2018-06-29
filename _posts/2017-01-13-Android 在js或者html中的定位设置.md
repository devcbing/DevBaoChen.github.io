---
layout: post
title: Android 在js或者html中的定位设置
categories: [Android, ]
description: Android 在js或者html中的定位设置.md
keywords: Android, 
---

### Android 在js或者html中的定位设置

首先需要使WebView支持js的调用
```
  webView.getSettings().setJavaScriptEnabled(true);
```
当然，在Android中需要有定位权限和访问网络的权限
AndroidManifest.xml
```
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.INTERNET" />
```
WebView需要是设置定位路径保存的地址
因为Geolocation需要用数据库来缓存定位信息和相关权限设置信息,所以设置定位信息缓存路径.如果不设置缓存路径的话,缓存的存储将不会被自动分配.
```
webView.getSettings().setGeolocationDatabasePath( context.getFilesDir().getPath() );
```
并且WebView需要使用WebChromeClient，因为WebChromeClient中有下面的方法：
```
webView.setWebChromeClient(new WebChromeClient() {
 public void onGeolocationPermissionsShowPrompt(String origin, GeolocationPermissions.Callback callback) {
    callback.invoke(origin, true, false);
 }
});
```
该方法被WebView回调，以获取定位信息并传给JavaScript


