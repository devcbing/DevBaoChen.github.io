---
layout: post
title: Flutter 布局详解
categories: [Flutter, ]
description: Flutter 布局详解
keywords: Flutter, 
---

由于 Flutter 在 UI 界面上，`一切皆为 widget` ,所以 widget 在flutter 十分重要，本文主要介绍 Flutter 的 布局介绍 ，主要的介绍就是 widget 。

![](/images/posts/flutter/2018-09-30-00.jpeg)
### Flutter 布局特性

#### 什么是边界约束

Flutter中的边界约束，是指widget可以按照指定限定条件，来决定自身如何占用布局空间。由于 Flutter 从React 中 借鉴了一些东西，包括布局思想，但是没有像 React 那样抽离出样式。而是实现了许多不同的 widget。使用这些不同的 widget 可以像搭积木一样，实现不同的布局效果。 写法和 React 差不多，但是没有把样式单独提取出来。

这样做的优点，可以将布局限定在有限的范围内，使得在完成布局的同时， 让整体效果可以统一渲染，加快了编译速度。

但是同样存在缺点，由于约束了 widget ，使得灵活性减少，而且不同的效果需要不同的 widget ，这样就会存在非常多的 widget 需要开发人员去掌握。

#### 约束的种类

在 Flutter 中，widget 是由底层的 RenderBox 来渲染的，渲染的边界约束 (Constraints) 由其父级给出，widget 由于这些约束存在，所以可以调节其自身尺寸， 约束包含最小最大宽高，尺寸则是具体的宽和高。

在 Android 原生实现的布局中，存在 match_parent、wrap_content 和自定义的尺寸来对布局进行约束。
同样的在 Flutter 中也存在这三种约束，只是写法不一样而已。

- 尽可能大的约束： 比如： Center、ListView 等等；
- 跟随自身大小的约束： 比如： Transform、Opacity 等等；
- 指定尺寸的约束： 比如： Image、 Text 等等。

但是，Flutter 并没有把 widget 处理的这么绝对，这些约束条件包含在 widget 里，不像 Android 可以在外面去指定。因此，一些 widget，例如 Container，会根据参数的不同，约束条件也不同。Container 默认是尽可能大的，但是给定尺寸的话，就会优先使用具体值。不同的 widget 可能设置条件不同、其子 widget 不同，约束条件也会不一样。Flutter 将每种 widget 限制在不同的约束范围里，实际布局的时候，还需要综合去考虑。 

### widget 的分类：

分类方式： 通过 子元素的个数来进行分类，可以分为：

- 单个子元素（child）布局： 目前包含 Container、Padding等等18种；
- 多个子元素 (children) 布局: 包含 Row、Column等11种。
- ayout helper： 比如 ListView.Builder，在元素过多的布局中，使用这种方式会更加高效。类似Android 中的 RecyclerView，有自动的回收机制。这种严格意义上不能算是一个种类。

#### 使用 widget 这种方式的优缺点

优点：使用这种方式可以统一渲染、更新布局效果，使得开发更加高效。
缺点：由于 Flutter 提供了太多的 widget ，导致学习成本变高，并且，很多 widget 都可以实现相同的 UI 效果，所以存在实现某一 UI 对 widget 选择的困扰。

### Widget 详解

在 Flutter 中，我们平时自定义的 widget，一般都是继承自 StatefulWidget 或 StatelessWidget （并不是只有这两种），这两种 widget 也是目前最常用的两种。如果一个控件自身状态不会去改变，创建了就直接显示，不会有色值、大小或者其他属性的变化，这种 widget 一般都是继承自 StatelessWidget ，常见的有 Container、ScrollView 等。如果一个控件需要动态的去改变或者相应一些状态，例如点击态、色值、内容区域等，那么一般都是继承自 StatefulWidget，常见的有 CheckBox、AppBar、TabBar 等。其实单纯的从名字也可以看出这两种 widget 的区别，这两种 widget 都是继承自 Widget 类。

#### Widget类

Widget 类在 Flutter 中是非常重要的，继承自 Widget 类的有 PreferredSizeWidget、ProxyWidget、RenderObjectWidget、StatefulWidget、StatelessWidget。我们日常使用的绝大部分 widget 都是继承自Widget类，

从 Widget 的源码中可以看到下面的构造函数： 

```
const Widget({ this.key });
final Key key;
```

其中的 key 是用来控制 widget 树中替换 widget 的时候使用的。 其中 Key 类是 Widget 、Element 以及SemanticsNode 的唯一标识符，继承自 Key 的还有 LocalKey 以及 GlobalKey 。

#### State 

State 的作用:
- 1.在 widget 构建的时候可以被同步读取；
- 2.在 widget 生命周期中可能会被改变。

##### State 的生命周期

State 的生命周期有四种状态：

- created: 当 State 对象被创建的时候，State.initState 方法会被调用；
- initialized： 当 State 对象被创建，但还没有准备构建时， State.didChangeDependencies 在这个时候会被调用;
- ready： State 对象已经准备好了构建， State.dispose 没有被调用的时候；
- defunct: State.dispose 被调用后， State 对象不能够被创建。

![](/images/posts/flutter/2018-10-02-01.png)

完整生命周期如下：
- 创建一个 State 对象时，会调用 StatefulWidget.createState；
- 和一个 BuildContext 相关联，可以认为被加载了（mounted）；
- 调用 initState；
- 调用 didChangeDependencies；
- 经过上述步骤，State 对象被完全的初始化了，调用 build；
- 如果有需要，会调用 didUpdateWidget；
- 如果处在开发模式，热加载会调用 reassemble；
- 如果它的子树（subtree）包含需要被移除的 State 对象，会调用 deactivate；
- 调用 dispose , State 对象以后都不会被构建；
- 当调用了 dispose , State 对象处于未加载（unmounted），已经被 dispose 的 State 对象没有办法被重新加载（remount）。

#### setState

State 的中有一个比较重要的方法就是 setState ， 当状态被修改时，widget 会被更新，比如点击 CheckBox ,会出现选中和非选中之间切换，就是通过修改状态来改变的。

通过 setState 的源码可以知道，出现下面的情况会抛出异常；
- 传入null；
- 处在 defunct 阶段修改状态；
- created 阶段还没有被加载 (mounted);
- 参数返回一个 Future 对象。

检查完一系列异常后，最后调用代码如下：

```
_element.markNeedsBuild();
```  

markNeedsBuild 内部，则是通过标记 element 为 diry ，在下一帧的时候重建（rebuild）。可以看出setState 并不是立即生效，它只是将 widget 进行了标记，真正的 rebuild 操作，则是等到下一帧的时候才会去进行。

#### StatefulWidget 和 StatelessWidget

![](/images/posts/flutter/2018-10-02-02.png)

一个 StatelessWidget 可以用多个不同的 BuildContext 构建，而一个 StatefulWidget 会为每个   BuildContext 创建一个 State 对象。

##### StatelessWidget

对于StatelessWidget，build方法会在如下三种情况下调用，

- 1.widget 第一次被插入到树中；
- 2.widget 的父节点更改了配置（configuration）；
- 3.widget 依赖的 InheritedWidget 改变了。

```
class GreenFrog extends StatelessWidget {
  const GreenFrog({ Key key }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return new Container(color: const Color(0xFF2DBD3A));
  }
}
```

##### StatefulWidget
StatefulWidget的两个主要类别：

- 1.在 initState 中创建资源，在 dispose 中销毁，但是不依赖于 InheritedWidget 或者调用setState 方法，这类 widget 基本上用在一个应用或者页面的 root ；
- 2.使用 setState 或者依赖于 InheritedWidget ，这种在营业生命周期中会被重建（rebuild）很多次。

```
class YellowBird extends StatefulWidget {
  const YellowBird({ Key key }) : super(key: key);

  @override
  _YellowBirdState createState() => new _YellowBirdState();
}

class _YellowBirdState extends State<YellowBird> {
  @override
  Widget build(BuildContext context) {
    return new Container(color: const Color(0xFFFFE306));
  }
}
```

 


 





