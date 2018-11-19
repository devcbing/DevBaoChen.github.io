---
layout: post
title: ListBody、ListView、CustomMultiChildLayout详解
categories: [Flutter, ]
description: ListBody、ListView、CustomMultiChildLayout详解
keywords: Flutter, 
---

ListBody、ListView、CustomMultiChildLayout详解


![](/images/posts/flutter/2018-09-30-00.jpeg)


### ListBody

ListBody 是一个不常直接使用的控件，一般都会配合 ListView 或者 Column 等控件使用。 ListBody 的作用是按给定的轴方向，按照顺序排列子节点。


#### ListBody 布局

在主轴上，子节点按照顺序进行布局，在交叉轴上，子节点尺寸会被拉伸，以适应交叉轴的区域。

在主轴上，给予子节点的空间必须是不受限制的（ unlimited ），使得子节点可以全部被容纳， ListBody 不会去裁剪或者缩放其子节点。

#### ListBody 继承关系

```

Object > Diagnosticable > DiagnosticableTree > Widget > RenderObjectWidget > MultiChildRenderObjectWidget > ListBody

```

#### ListBody 示例

```

Flex(
  direction: Axis.vertical,
  children: <Widget>[
    ListBody(
      mainAxis: Axis.vertical,
      reverse: false,
      children: <Widget>[
        Container(color: Colors.red, width: 50.0, height: 50.0,),
        Container(color: Colors.yellow, width: 50.0, height: 50.0,),
        Container(color: Colors.green, width: 50.0, height: 50.0,),
        Container(color: Colors.blue, width: 50.0, height: 50.0,),
        Container(color: Colors.black, width: 50.0, height: 50.0,),
      ],
  )],
)


```

#### ListBody 源码

##### 构造函数

```

ListBody({
  Key key,
  this.mainAxis = Axis.vertical,
  this.reverse = false,
  List<Widget> children = const <Widget>[],
})

```

##### 属性解析

`mainAxis `：排列的主轴方向。
`reverse `：是否反向。


##### 源码

ListBody 的布局代码非常简单，根据主轴的方向，对子节点依次排布。

当向右的时候，布局代码如下，向下的代码类似

```

double mainAxisExtent = 0.0;
RenderBox child = firstChild;
switch (axisDirection) {
case AxisDirection.right:
  final BoxConstraints innerConstraints = new BoxConstraints.tightFor(height: constraints.maxHeight);
  while (child != null) {
    child.layout(innerConstraints, parentUsesSize: true);
    final ListBodyParentData childParentData = child.parentData;
    childParentData.offset = new Offset(mainAxisExtent, 0.0);
    mainAxisExtent += child.size.width;
    assert(child.parentData == childParentData);
    child = childParentData.nextSibling;
  }
  size = constraints.constrain(new Size(mainAxisExtent, constraints.maxHeight));
  break;
}

```

当向左的时候，布局代码如下，向上的代码类似：

```

double mainAxisExtent = 0.0;
RenderBox child = firstChild;
case AxisDirection.left:
  final BoxConstraints innerConstraints = new BoxConstraints.tightFor(height: constraints.maxHeight);
  while (child != null) {
    child.layout(innerConstraints, parentUsesSize: true);
    final ListBodyParentData childParentData = child.parentData;
    mainAxisExtent += child.size.width;
    assert(child.parentData == childParentData);
    child = childParentData.nextSibling;
  }
  double position = 0.0;
  child = firstChild;
  while (child != null) {
    final ListBodyParentData childParentData = child.parentData;
    position += child.size.width;
    childParentData.offset = new Offset(mainAxisExtent - position, 0.0);
    assert(child.parentData == childParentData);
    child = childParentData.nextSibling;
  }
  size = constraints.constrain(new Size(mainAxisExtent, constraints.maxHeight));
  break;

```

向右或者向下的时候，布局代码很简单，依次去排列。当向左或者向上的时候，首先会去计算主轴所占的空间，然后再去计算每个节点的位置。



### ListView

ListView 是一个非常常用的控件，涉及到数据列表展示的，一般情况下都会选用该控件。 ListView 跟 GridView 相似，基本上是一个  slivers 里面只包含一个 SliverList 的 CustomScrollView 。

#### ListView 布局

```

ListView 在主轴方向可以滚动，在交叉轴方向，则是填满 ListView 。

```

#### ListView 继承关系

```
Object > Diagnosticable > DiagnosticableTree > Widget > StatelessWidget > ScrollView > BoxScrollView > ListView

```

这是一个组合控件。 ListView 跟 GridView 类似，都是继承自 BoxScrollView 。

#### ListView 示例代码

```

ListView(
  shrinkWrap: true,
  padding: EdgeInsets.all(20.0),
  children: <Widget>[
    Text('I\'m dedicating every day to you'),
    Text('Domestic life was never quite my style'),
    Text('When you smile, you knock me out, I fall apart'),
    Text('And I thought I was so smart'),
  ],
)

ListView.builder(
  itemCount: 1000,
  itemBuilder: (context, index) {
    return ListTile(
      title: Text("$index"),
    );
  },
)


```

两个示例都是官方文档上的例子，第一个展示四行文字，第二个展示 1000 个 item 。


#### ListView 源码

##### 构造函数

```

ListView({
  Key key,
  Axis scrollDirection = Axis.vertical,
  bool reverse = false,
  ScrollController controller,
  bool primary,
  ScrollPhysics physics,
  bool shrinkWrap = false,
  EdgeInsetsGeometry padding,
  this.itemExtent,
  bool addAutomaticKeepAlives = true,
  bool addRepaintBoundaries = true,
  double cacheExtent,
  List<Widget> children = const <Widget>[],
})

```

同时也提供了如下额外的三种构造方法，方便开发者使用。

```

ListView.builder
ListView.separated
ListView.custom

```

##### 属性解析

ListView 大部分属性同 GridView ，想了解的读者可以看一下之前所写的 GridView 相关的文章。这里只介绍一个属性

`itemExtent `： ListView 在滚动方向上每个 item 所占的高度值。

##### 源码

```

@override
Widget buildChildLayout(BuildContext context) {
  if (itemExtent != null) {
    return new SliverFixedExtentList(
      delegate: childrenDelegate,
      itemExtent: itemExtent,
    );
  }
  return new SliverList(delegate: childrenDelegate);
}

```

ListView 标准构造布局代码如上所示，底层是用到的 SliverList 去实现的。 ListView 是一个 slivers 里面只包含一个 SliverList  的 CustomScrollView 。源码这块儿可以参考 GridView 


#### ListView 使用场景

一般涉及到列表展示的，一般都会选择 ListView 。

但是需要注意一点， ListView 的标准构造函数适用于数目比较少的场景，如果数目比较多的话，最好使用 `ListView.builder `。

ListView 的标准构造函数会将所有 item 一次性创建，而 `ListView.builder ` 会创建滚动到屏幕上显示的 item 。


### CustomMultiChildLayout

单节点布局控件中介绍过一个类似的控件-- CustomSingleChildLayout ，都是通过 delegate 去实现自定义布局，只不过这次是多节点的自定义布局的控件，通过提供的 delegate ，可以实现控制节点的位置以及尺寸。


#### CustomMultiChildLayout 布局

CustomMultiChildLayout 提供的 delegate 可以控制子节点的布局，具体在如下几点：

 - 1.可以决定每个子节点的布局约束条件；
 - 2.可以决定每个子节点的位置；
 - 3.可以决定自身的尺寸，但是自身的自身必须不能依赖子节点的尺寸。

可以看到，跟 CustomSingleChildLayout 的 delegate 提供的作用类似，只不过 CustomMultiChildLayout 的稍微会复杂点。

#### CustomMultiChildLayout 继承关系

```

Object > Diagnosticable > DiagnosticableTree > Widget > RenderObjectWidget > MultiChildRenderObjectWidget > CustomMultiChildLayout

```

#### CustomMultiChildLayout 示例

```

class TestLayoutDelegate extends MultiChildLayoutDelegate {
  TestLayoutDelegate();

  static const String title = 'title';
  static const String description = 'description';

  @override
  void performLayout(Size size) {
    final BoxConstraints constraints =
        new BoxConstraints(maxWidth: size.width);

    final Size titleSize = layoutChild(title, constraints);
    positionChild(title, new Offset(0.0, 0.0));

    final double descriptionY = titleSize.height;
    layoutChild(description, constraints);
    positionChild(description, new Offset(0.0, descriptionY));
  }

  @override
  bool shouldRelayout(TestLayoutDelegate oldDelegate) => false;
}

Container(
  width: 200.0,
  height: 100.0,
  color: Colors.yellow,
  child: CustomMultiChildLayout(
    delegate: TestLayoutDelegate(),
    children: <Widget>[
      LayoutId(
        id: TestLayoutDelegate.title,
        child: new Text("This is title",
            style: TextStyle(fontSize: 20.0, color: Colors.black)),
      ),
      LayoutId(
        id: TestLayoutDelegate.description,
        child: new Text("This is description",
            style: TextStyle(fontSize: 14.0, color: Colors.red)),
      ),
    ],
  ),
)

```

上面的 TestLayoutDelegate 作用很简单，对子节点进行尺寸以及位置调整。可以看到，每一个子节点必须用一个 LayoutId 控件包裹起来，在 delegate 中可以对不同 id 的控件进行调整 。

#### CustomMultiChildLayout 源码

##### 构造函数

```

CustomMultiChildLayout({
  Key key,
  @required this.delegate,
  List<Widget> children = const <Widget>[],
})

```

##### 属性解析

`delegate `：对子节点进行尺寸以及位置调整的 delegate 。

 
##### 源码

```
@override
void performLayout() {
  size = _getSize(constraints);
  delegate._callPerformLayout(size, firstChild);
}

```
CustomMultiChildLayout 的布局代码很简单，调用 delegate 中的布局函数进行相关的操作，本身做的处理很少，在这里不做过多的解释。





