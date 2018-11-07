---
layout: post
title:  Stack、IndexedStack、GridView详解
categories: [Flutter, ]
description:  Stack、IndexedStack、GridView详解
keywords: Flutter, 
---

Stack、IndexedStack、GridView详解


![](/images/posts/flutter/2018-09-30-00.jpeg)


### Stack


Stack 可以类比 web 中的 absolute ，绝对布局。绝对布局一般在移动端开发中用的较少，但是在某些场景下，还是有其作用。当然，能用 Stack 绝对布局完成的，用其他控件组合也都能实现。

#### Stack 布局行为

 Stack 的布局行为，根据 child 是 positioned 还是 non-positioned 来区分。
 
  - 1.对于 positioned 的子节点，它们的位置会根据所设置的 top 、 bottom 、 right 以及 left 属性来确定，这几个值都是相对于  Stack 的左上角；

 - 2.对于 non-positioned 的子节点，它们会根据 Stack 的 aligment 来设置位置。


对于绘制 child 的顺序，则是第一个 child 被绘制在最底端，后面的依次在前一个 child 的上面，类似于 web 中的 z-index 。如果想调整显示的顺序，则可以通过摆放 child 的顺序来进行。


#### Stack 继承关系

```
Object > Diagnosticable > DiagnosticableTree > Widget > RenderObjectWidget > MultiChildRenderObjectWidget > Stack

```

#### Stack 示例

```
Stack(
  alignment: const Alignment(0.6, 0.6),
  children: [
    CircleAvatar(
      backgroundImage: AssetImage('images/pic.jpg'),
      radius: 100.0,
    ),
    Container(
      decoration: BoxDecoration(
        color: Colors.black45,
      ),
      child: Text(
        'Mia B',
        style: TextStyle(
          fontSize: 20.0,
          fontWeight: FontWeight.bold,
          color: Colors.white,
        ),
      ),
    ),
  ],
);

```
得到的效果如下

![](/images/posts/flutter/2018-10-10-01.png)


#### Stack 源码


##### 构造函数

```
Stack({
  Key key,
  this.alignment = AlignmentDirectional.topStart,
  this.textDirection,
  this.fit = StackFit.loose,
  this.overflow = Overflow.clip,
  List<Widget> children = const <Widget>[],
})

```

##### 属性解析

`alignment `：对齐方式，默认是左上角（ topStart ）。

`textDirection `：文本的方向，绝大部分不需要处理。

`fit `：定义如何设置 non-positioned 节点尺寸，默认为 loose 。


其中 StackFit 有如下几种：

 - 1.loose ：子节点宽松的取值，可以从 min 到 max 的尺寸；

 - 2.expand ：子节点尽可能的占用空间，取 max 尺寸；

 - 3.passthrough ：不改变子节点的约束条件。


`overflow `：超过的部分是否裁剪掉（ clipped ）。

##### 源码

 - 1.如果不包含子节点，则尺寸尽可能大。

```

if (childCount == 0) {
  size = constraints.biggest;
  return;
}

```

 - 2.根据 fit 属性，设置 non-positioned 子节点约束条件。

```
switch (fit) {
  case StackFit.loose:
    nonPositionedConstraints = constraints.loosen();
    break;
  case StackFit.expand:
    nonPositionedConstraints = new BoxConstraints.tight(constraints.biggest);
    break;
  case StackFit.passthrough:
    nonPositionedConstraints = constraints;
    break;
}

```

 - 3.对 non-positioned 子节点进行布局。

```

RenderBox child = firstChild;
while (child != null) {
  final StackParentData childParentData = child.parentData;
  if (!childParentData.isPositioned) {
    hasNonPositionedChildren = true;
    child.layout(nonPositionedConstraints, parentUsesSize: true);
    final Size childSize = child.size;
    width = math.max(width, childSize.width);
    height = math.max(height, childSize.height);
  }
  child = childParentData.nextSibling;
}

```

 - 4.根据是否包含 positioned 子节点，对 stack 进行尺寸调整。

```

if (hasNonPositionedChildren) {
  size = new Size(width, height);
} else {
  size = constraints.biggest;
}

```

 - 5.最后对子节点位置的调整，这个调整过程中，则根据 alignment 、 positioned 节点的绝对位置等信息，对子节点进行布局。

第一步是根据 positioned 的绝对位置，计算出约束条件后进行布局。

```

if (childParentData.left != null && childParentData.right != null)
  childConstraints = childConstraints.tighten(width: size.width - childParentData.right - childParentData.left);
else if (childParentData.width != null)
  childConstraints = childConstraints.tighten(width: childParentData.width);

if (childParentData.top != null && childParentData.bottom != null)
  childConstraints = childConstraints.tighten(height: size.height - childParentData.bottom - childParentData.top);
else if (childParentData.height != null)
  childConstraints = childConstraints.tighten(height: childParentData.height);

child.layout(childConstraints, parentUsesSize: true);

```

第二步则是位置的调整，其中坐标的计算如下：

```

double x;
if (childParentData.left != null) {
  x = childParentData.left;
} else if (childParentData.right != null) {
  x = size.width - childParentData.right - child.size.width;
} else {
  x = _resolvedAlignment.alongOffset(size - child.size).dx;
}

if (x < 0.0 || x + child.size.width > size.width)
  _hasVisualOverflow = true;

double y;
if (childParentData.top != null) {
  y = childParentData.top;
} else if (childParentData.bottom != null) {
  y = size.height - childParentData.bottom - child.size.height;
} else {
  y = _resolvedAlignment.alongOffset(size - child.size).dy;
}

if (y < 0.0 || y + child.size.height > size.height)
  _hasVisualOverflow = true;

childParentData.offset = new Offset(x, y);

```


#### Stack 使用场景

Stack  的场景还是比较多的，对于需要叠加显示的布局，一般都可以使用 Stack 。有些场景下，也可以被其他控件替代，我们应该选择开销较小的控件去实现。


### IndexedStack

IndexedStack  继承自 Stack ，它的作用是显示第 index 个 child ，其他 child 都是不可见的。所以   IndexedStack  的尺寸永远是跟最大的子节点尺寸一致。

#### IndexedStack 示例

将 Stack 的例子稍加改造，将 index 设置为 1 ，也就是显示含文本的 Container 的节点。

```
Container(
  color: Colors.yellow,
  child: IndexedStack(
    index: 1,
    alignment: const Alignment(0.6, 0.6),
    children: [
      CircleAvatar(
        backgroundImage: AssetImage('images/pic.jpg'),
        radius: 100.0,
      ),
      Container(
        decoration: BoxDecoration(
          color: Colors.black45,
        ),
        child: Text(
          'Mia B',
          style: TextStyle(
            fontSize: 20.0,
            fontWeight: FontWeight.bold,
            color: Colors.white,
          ),
        ),
      ),
    ],
  ),
)
```

![](/images/posts/flutter/2018-10-10-02.png)


#### IndexedStack 源码


其绘制代码很简单，因为继承自 Stack ，布局方面表现基本一致，不同之处在于其绘制的时候，只是将第 Index个child 进行了绘制。

```

@override
void paintStack(PaintingContext context, Offset offset) {
if (firstChild == null || index == null)
  return;
final RenderBox child = _childAtIndex();
final StackParentData childParentData = child.parentData;
context.paintChild(child, childParentData.offset + offset);
}

```

#### IndexedStack 使用场景

如果需要展示一堆控件中的一个，可以使用 IndexedStack 。

### GridView

GridView  在移动端上非常的常见，就是一个滚动的多列列表。

#### GridView 布局

GridView 本身是尽量占满空间区域，布局行为上完全继承自 ScrollView 。

#### GridView 继承关系

```
Object > Diagnosticable > DiagnosticableTree > Widget > StatelessWidget > ScrollView > BoxScrollView > GridView

```

所以 GridView 是在 ScrollView 的基础上封装的。

#### GridView  示例代码

```

GridView.count(
  crossAxisCount: 2,
  children: List.generate(
    100,
    (index) {
      return Center(
        child: Text(
          'Item $index',
          style: Theme.of(context).textTheme.headline,
        ),
      );
    },
  ),
);

```

可参考 [https://flutter.io/docs/cookbook/lists/grid-lists](https://flutter.io/docs/cookbook/lists/grid-lists)


#### GridView 源码

##### 构造函数

```
GridView({
  Key key,
  Axis scrollDirection = Axis.vertical,
  bool reverse = false,
  ScrollController controller,
  bool primary,
  ScrollPhysics physics,
  bool shrinkWrap = false,
  EdgeInsetsGeometry padding,
  @required this.gridDelegate,
  bool addAutomaticKeepAlives = true,
  bool addRepaintBoundaries = true,
  double cacheExtent,
  List<Widget> children = const <Widget>[],
})

```

同时也提供了如下额外的四种构造方法，方便开发者使用。

```
GridView.builder
GridView.custom
GridView.count
GridView.extent

```

##### 属性解析

`scrollDirection `：滚动的方向，有垂直和水平两种，默认为垂直方向（ Axis.vertical ）。

`reverse `：默认是从上或者左向下或者右滚动的，这个属性控制是否反向，默认值为 false ，不反向滚动。

`controller `：控制 child 滚动时候的位置。

`primary `：是否是与父节点的 PrimaryScrollController 所关联的主滚动视图。

`physics `：滚动的视图如何响应用户的输入。

`shrinkWrap `：滚动方向的滚动视图内容是否应该由正在查看的内容所决定。

`padding `：四周的空白区域。

`gridDelegate `：控制 GridView 中子节点布局的 delegate 。

`cacheExtent `：缓存区域。


##### 源码

```

@override
Widget build(BuildContext context) {
  final List<Widget> slivers = buildSlivers(context);
  final AxisDirection axisDirection = getDirection(context);

  final ScrollController scrollController = primary
    ? PrimaryScrollController.of(context)
    : controller;
  final Scrollable scrollable = new Scrollable(
    axisDirection: axisDirection,
    controller: scrollController,
    physics: physics,
    viewportBuilder: (BuildContext context, ViewportOffset offset) {
      return buildViewport(context, offset, axisDirection, slivers);
    },
  );
  return primary && scrollController != null
    ? new PrimaryScrollController.none(child: scrollable)
    : scrollable;
}

```

上面这段代码是 ScrollView 的 build 方法， GridView 就是一个特殊的 ScrollView 。 GridView 本身代码没有什么，基本上都是 ScrollView 上的东西，主要会涉及到 Scrollable 、 Sliver 、 Viewport 等内容，这些内容比较多，这里先省略。

#### GridView 使用场景

GridView 实际上是一个 silvers 只包含一个 SilverGrid 的 CustomScrollView 。 所以在很多场景中都会用到。


