---
layout: post
title: Dart async/await
categories: [Dart, ]
description: Dart async/await
keywords: Dart, 
---

Dart async/await


<a name="11yMz"></a>
### 异步方法
在 dart 中 ，要实现异步方法，只需要在方法后面增加 async  关键字

```dart
//从网络获取数据
Future<String> getData(String url) async{
  var data = "data"; // 从网络获得的数据
  return data;  //将数据返回。
}
```

dart 中异步方法返回的一定是一个  `Future`  类型的，表示这个方法一定会在未来运行。<br />上面的异步方法是耗时的，如果想要调用上面的异步方法，需要使用 await 

```dart
Future<void> useData() async{
  var data = await getData("url");
}
```

<a name="Km8E6"></a>
### 异步Future 等待
当我们希望在前一个异步方法结束后，再执行下一个异步方法时，可以使用 `Future.``_wait_``()`  来实现。

```dart
Future doThing1() async{
  print("doThing1");

}
Future doThing2() async{
  print("doThing2");
}
Future doThing3() async{
  print("doThing3");
}

//使用 
 Future.wait([doThing1(),doThing2(),doThing3()]);
```

