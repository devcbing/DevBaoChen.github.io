---
layout: post
title: Flutter 框架概览
categories: [Flutter, ]
description: Flutter 是一款移动应用程序 SDK，一份代码可以同时生成 iOS 和 Android 两个高性能、高保真的应用程序。
keywords: Flutter, 
---

Flutter 是一款移动应用程序 SDK ，一份代码可以同时生成 iOS 和 Android 两个高性能、高保真的应用程序。
Flutter 的目标是: 让开发人员能够在不同的系统平台中都能实现流畅、高性能的应用程序，所以在 滚动、排版、图标等做了相应的兼容性。


![](/images/posts/flutter/2018-09-30-00.jpeg)


### Flutter 框架
Flutter 框架是一个分层的结构，每个层都建立在前一层之上。

![](/images/posts/flutter/2018-09-30-03.png)

Flutter框架分为 Framework 和 Engine 两部分，
- Framework 提供了各种基础组件 
- Engine 主要是渲染各种 widget

有关构成 Flutter 分层框架的完整库，请参阅[API文档](https://docs.flutter.io/)。


### Flutter 核心原则
Flutter 包含一个现代的响应式框架、一个 2D 渲染引擎、还有丰富的 widget、丰富的开发工具。 利用这些东西，可以让你很方便的对你的应用程序进行设计、构建、开发、调试和测试。


#### 一切皆为 widget
Widget 是 Flutter 界面 UI 的基本元素。每个 Widget 都是用户界面的一个不可变声明。同时与其他将视图、控制器、布局和其他属性分离的框架不同， Flutter 具有一直的、统一的对象模型： widget。使得在更新 widget 的时候，框架能够更加的高效。

widget 可以被定义为：

- 一个元素结构（比如按钮，菜单等）
- 一个文本样式（比如字体和颜色的解决方案）
- 布局的一个方面 (比如填充)
- 其他...

Widget 根据布局形成一个层次结构。每个 widget 嵌入其中，并继承其父项的属性。没有单独的“应用程序”对象，相反，根   widget 扮演着这个角色。

您可以通过告诉框架使用另一个widget替换层次结构中的widget来响应事件，例如用户交互，替换后框架会比较新的和旧的widget，并高效地更新用户界面。

`组合>集成`

Widget 本身通常由许多更小的、单一用途 widget 组成，这些 widget 结合起来产生强大的效果。例如，Container 是一个常用的 widget，由多个widget组成，这些widget负责布局、绘制、定位和调整大小。具体来说，Container 由 LimitedBox、 ConstrainedBox、 Align、 Padding、 DecoratedBox、 和 Transform 组成。 您可以用各种方式组合这些以及其他简单的 widget，而不是继承容器。

Widget 的类层次结构很浅且很宽，可以最大限度地增加可能的组合数量

![](/images/posts/flutter/2018-09-30-02.png)

基于这种 widget 架构，你可以使用很小的 widget 来组成一个大的 widget，并实现你自己优美的、流畅的应用界面。

### Flutter 跨平台的解决方案

![](/images/posts/flutter/2018-09-30-04.png)

Flutter 跨平台最核心的部分，是它的高性能渲染引擎（Flutter Engine）。Flutter 不使用浏览器技术，也不使用 Native 的原生控件，它使用自己的渲染引擎来绘制 widget。

对于 Android 平台，Flutter 引擎的 C/C++ 代码是由 NDK 编译，在 iOS 平台，则是由 LLVM 编译，两个平台的 Dart 代码都是 AOT 编译为本地代码，Flutter 应用程序使用本机指令集运行。
Flutter 正是是通过使用相同的渲染器、框架和一组 widget ，来同时构建 iOS 和 Android 应用，而无需维护两套独立的代码库。

### 参考：
[https://flutterchina.club/technical-overview/](https://flutterchina.club/technical-overview/)

