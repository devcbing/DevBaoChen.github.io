---
layout: post
title:  Row、Column详解
categories: [Flutter, ]
description:  Row、Column详解
keywords: Flutter, 
---

Row、Column详解


![](/images/posts/flutter/2018-09-30-00.jpeg)

###  Row

在Flutter中非常常见的一个多子节点控件，将children排列成一行。 但是 自身不带滚动属性，在 debug 下面会显示溢出。

#### Row 布局

Row 的布局有六个步骤，这种布局表现来自 Flex （ Row 和 Column 的父类）：

- 1.首先按照不受限制的主轴（main axis）约束条件，对 flex 为 null 或者为 0 的 child 进行布局，然后按照交叉轴（ cross axis）的约束，对 child 进行调整；

- 2.按照不为空的 flex 值，将主轴方向上剩余的空间分成相应的几等分；

- 3.对上述步骤 flex 值不为空的 child ，在交叉轴方向进行调整，在主轴方向使用最大约束条件，让其占满步骤 2 所分得的空间；

- 4.Flex 交叉轴的范围取自子节点的最大交叉轴；

- 5.主轴 Flex 的值是由 mainAxisSize 属性决定的，其中 MainAxisSize 可以取 max 、 min 以及具体的 value 值；

- 6.每一个 child 的位置是由 mainAxisAlignment 以及 crossAxisAlignment 所决定。

![](/images/posts/flutter/2018-10-09-01.png)

#### Row 继承关系

```
Object > Diagnosticable > DiagnosticableTree > Widget > RenderObjectWidget > MultiChildRenderObjectWidget > Flex > Row

```
Row 以及 Column 都是 Flex 的子类，它们的具体实现也都是由 Flex 完成，只是参数不同。

#### Row 示例

```

Row(
  children: <Widget>[
    Expanded(
      child: Container(
        color: Colors.red,
        padding: EdgeInsets.all(5.0),
      ),
      flex: 1,
    ),
    Expanded(
      child: Container(
        color: Colors.yellow,
        padding: EdgeInsets.all(5.0),
      ),
      flex: 2,
    ),
    Expanded(
      child: Container(
        color: Colors.blue,
        padding: EdgeInsets.all(5.0),
      ),
      flex: 1,
    ),
  ],
)

```

使用 Expanded 控件，将一行的宽度分成四个等分，第一、三个 child 占1/4的区域，第二个 child 占1/2区域，由 flex 属性控制。


#### Row 源码

##### 构造函数

```
Row({
  Key key,
  MainAxisAlignment mainAxisAlignment = MainAxisAlignment.start,
  MainAxisSize mainAxisSize = MainAxisSize.max,
  CrossAxisAlignment crossAxisAlignment = CrossAxisAlignment.center,
  TextDirection textDirection,
  VerticalDirection verticalDirection = VerticalDirection.down,
  TextBaseline textBaseline,
  List<Widget> children = const <Widget>[],
})

```

##### 属性解析

`MainAxisAlignment ` : 主轴方向上的对齐方式，会对child的位置起作用，默认是start。

其中MainAxisAlignment枚举值：

   - 1.center ：将 children 放置在主轴的中心；

   - 2.end ：将 children 放置在主轴的末尾；

   - 3.spaceAround ：将主轴方向上的空白区域均分，使得 children 之间的空白区域相等，但是首尾 child 的空白区域为1/2；

   - 4.spaceBetween ：将主轴方向上的空白区域均分，使得 children 之间的空白区域相等，首尾 child 都靠近首尾，没有间隙；

   - 5.spaceEvenly ：将主轴方向上的空白区域均分，使得 children 之间的空白区域相等，包括首尾 child ；

   - 6.start ：将 children 放置在主轴的起点；


其中 spaceAround 、 spaceBetween 以及 spaceEvenly 的区别，就是对待首尾 child 的方式。其距离首尾的距离分别是空白区域的 1/2、0、1。

`MainAxisSize `：在主轴方向占有空间的值，默认是 max 。

MainAxisSize的取值有两种：

   - 1.max：根据传入的布局约束条件，最大化主轴方向的可用空间；

   - 2.min：与max相反，是最小化主轴方向的可用空间；


`CrossAxisAlignment `： children 在交叉轴方向的对齐方式，与 MainAxisAlignment 略有不同。

CrossAxisAlignment 枚举值有如下几种：

   - 1.baseline ：在交叉轴方向，使得 children 的 baseline 对齐；

   - 2.center ： children 在交叉轴上居中展示；

   - 3.end ： children 在交叉轴上末尾展示；

   - 4.start ： children 在交叉轴上起点处展示；

   - 5.stretch ：让 children 填满交叉轴方向；


`TextDirection ` : 阿拉伯语系的兼容设置，一般无需处理。

`VerticalDirection ` : 定义了 children 摆放顺序，默认是 down 。

 VerticalDirection 枚举值有两种：

   - 1.down ：从 top 到 bottom 进行布局；

   - 2.up ： 从 bottom 到 top 进行布局。

top 对应 Row 以及 Column 的话，就是左边和顶部， bottom 的话，则是右边和底部。

`TextBaseline `：使用的 TextBaseline 的方式，有两种，前面已经介绍过。

##### 源码

Row 以及 Column 的源代码就一个构造函数，具体的实现全部在它们的父类 Flex 中。

关于Flex的构造函数


```

Flex({
  Key key,
  @required this.direction,
  this.mainAxisAlignment = MainAxisAlignment.start,
  this.mainAxisSize = MainAxisSize.max,
  this.crossAxisAlignment = CrossAxisAlignment.center,
  this.textDirection,
  this.verticalDirection = VerticalDirection.down,
  this.textBaseline,
  List<Widget> children = const <Widget>[],
})

```
Flex 的构造函数就比 Row 和 Column 的多了一个参数。 Row 跟 Column 的区别，正是这个 direction 参数的不同。当为  Axis.horizontal 的时候，则是 Row ，当为 Axis.vertical 的时候，则是 Column 。

我们来看下 Flex 的布局函数，由于布局函数比较多，因此分段来讲解：

```
while (child != null) {
  final FlexParentData childParentData = child.parentData;
  totalChildren++;
  final int flex = _getFlex(child);
  if (flex > 0) {
    totalFlex += childParentData.flex;
    lastFlexChild = child;
  } else {
    BoxConstraints innerConstraints;
    if (crossAxisAlignment == CrossAxisAlignment.stretch) {
      switch (_direction) {
        case Axis.horizontal:
          innerConstraints = new BoxConstraints(minHeight: constraints.maxHeight,
                                                maxHeight: constraints.maxHeight);
          break;
        case Axis.vertical:
          innerConstraints = new BoxConstraints(minWidth: constraints.maxWidth,
                                                maxWidth: constraints.maxWidth);
          break;
      }
    } else {
      switch (_direction) {
        case Axis.horizontal:
          innerConstraints = new BoxConstraints(maxHeight: constraints.maxHeight);
          break;
        case Axis.vertical:
          innerConstraints = new BoxConstraints(maxWidth: constraints.maxWidth);
          break;
      }
    }
    child.layout(innerConstraints, parentUsesSize: true);
    allocatedSize += _getMainSize(child);
    crossSize = math.max(crossSize, _getCrossSize(child));
  }
  child = childParentData.nextSibling;
}


```

上面这段代码，剔除了一些 assert 以及错误信息之类的代码，不影响实际的理解。

在布局的开始，首先会遍历一遍child，遍历的作用有两点：

 - 1.对于存在 flex 值的 child ，计算出 flex 的和，找到最后一个包含 flex 值的 child 。找到这个 child ，是因为主轴对齐方式，可能会对它的位置做调整，需要找出来；
 - 2.对于不包含 flex 的 child ，根据交叉轴方向的设置，对 child 进行调整。

```

final double freeSpace = math.max(0.0, (canFlex ? maxMainSize : 0.0) - allocatedSize);
if (totalFlex > 0 || crossAxisAlignment == CrossAxisAlignment.baseline) {
  final double spacePerFlex = canFlex && totalFlex > 0 ? (freeSpace / totalFlex) : double.nan;
  child = firstChild;
  while (child != null) {
    final int flex = _getFlex(child);
    if (flex > 0) {
      final double maxChildExtent = canFlex ? (child == lastFlexChild ? (freeSpace - allocatedFlexSpace) : spacePerFlex * flex) : double.infinity;
      double minChildExtent;
      switch (_getFit(child)) {
        case FlexFit.tight:
          assert(maxChildExtent < double.infinity);
          minChildExtent = maxChildExtent;
          break;
        case FlexFit.loose:
          minChildExtent = 0.0;
          break;
      }
      BoxConstraints innerConstraints;
      if (crossAxisAlignment == CrossAxisAlignment.stretch) {
        switch (_direction) {
          case Axis.horizontal:
            innerConstraints = new BoxConstraints(minWidth: minChildExtent,
                                                  maxWidth: maxChildExtent,
                                                  minHeight: constraints.maxHeight,
                                                  maxHeight: constraints.maxHeight);
            break;
          case Axis.vertical:
            innerConstraints = new BoxConstraints(minWidth: constraints.maxWidth,
                                                  maxWidth: constraints.maxWidth,
                                                  minHeight: minChildExtent,
                                                  maxHeight: maxChildExtent);
            break;
        }
      } else {
        switch (_direction) {
          case Axis.horizontal:
            innerConstraints = new BoxConstraints(minWidth: minChildExtent,
                                                  maxWidth: maxChildExtent,
                                                  maxHeight: constraints.maxHeight);
            break;
          case Axis.vertical:
            innerConstraints = new BoxConstraints(maxWidth: constraints.maxWidth,
                                                  minHeight: minChildExtent,
                                                  maxHeight: maxChildExtent);
            break;
        }
      }
      child.layout(innerConstraints, parentUsesSize: true);
      final double childSize = _getMainSize(child);
      allocatedSize += childSize;
      allocatedFlexSpace += maxChildExtent;
      crossSize = math.max(crossSize, _getCrossSize(child));
    }
    if (crossAxisAlignment == CrossAxisAlignment.baseline) {
      final double distance = child.getDistanceToBaseline(textBaseline, onlyReal: true);
      if (distance != null)
        maxBaselineDistance = math.max(maxBaselineDistance, distance);
    }
    final FlexParentData childParentData = child.parentData;
    child = childParentData.nextSibling;
  }
}

```

上面的代码段所做的事情也有两点：

  - 1.为包含 flex 的 child 分配剩余的空间
对于每份 flex 所对应的空间大小，它的计算方式如下：

```

final double freeSpace = math.max(0.0, (canFlex ? maxMainSize : 0.0) - allocatedSize);
final double spacePerFlex = canFlex && totalFlex > 0 ? (freeSpace / totalFlex) : double.nan;

```

其中，allocatedSize 是不包含 flex 所占用的空间。当每一份 flex 所占用的空间计算出来后，则根据交叉轴的设置，对包含 flex 的 child 进行调整。

  - 2.计算出 baseline 值

如果交叉轴的对齐方式为 baseline ，则计算出最大的 baseline 值，将其作为整体的 baseline 值。

```
switch (_mainAxisAlignment) {
  case MainAxisAlignment.start:
    leadingSpace = 0.0;
    betweenSpace = 0.0;
    break;
  case MainAxisAlignment.end:
    leadingSpace = remainingSpace;
    betweenSpace = 0.0;
    break;
  case MainAxisAlignment.center:
    leadingSpace = remainingSpace / 2.0;
    betweenSpace = 0.0;
    break;
  case MainAxisAlignment.spaceBetween:
    leadingSpace = 0.0;
    betweenSpace = totalChildren > 1 ? remainingSpace / (totalChildren - 1) : 0.0;
    break;
  case MainAxisAlignment.spaceAround:
    betweenSpace = totalChildren > 0 ? remainingSpace / totalChildren : 0.0;
    leadingSpace = betweenSpace / 2.0;
    break;
  case MainAxisAlignment.spaceEvenly:
    betweenSpace = totalChildren > 0 ? remainingSpace / (totalChildren + 1) : 0.0;
    leadingSpace = betweenSpace;
    break;
}

```

然后，就是将 child 在主轴方向上按照设置的对齐方式，进行位置调整。上面代码就是计算前后空白区域值的过程，可以看出   spaceBetween 、 spaceAround 以及 spaceEvenly 的差别。

```

double childMainPosition = flipMainAxis ? actualSize - leadingSpace : leadingSpace;
child = firstChild;
while (child != null) {
  final FlexParentData childParentData = child.parentData;
  double childCrossPosition;
  switch (_crossAxisAlignment) {
    case CrossAxisAlignment.start:
    case CrossAxisAlignment.end:
      childCrossPosition = _startIsTopLeft(flipAxis(direction), textDirection, verticalDirection)
                           == (_crossAxisAlignment == CrossAxisAlignment.start)
                         ? 0.0
                         : crossSize - _getCrossSize(child);
      break;
    case CrossAxisAlignment.center:
      childCrossPosition = crossSize / 2.0 - _getCrossSize(child) / 2.0;
      break;
    case CrossAxisAlignment.stretch:
      childCrossPosition = 0.0;
      break;
    case CrossAxisAlignment.baseline:
      childCrossPosition = 0.0;
      if (_direction == Axis.horizontal) {
        assert(textBaseline != null);
        final double distance = child.getDistanceToBaseline(textBaseline, onlyReal: true);
        if (distance != null)
          childCrossPosition = maxBaselineDistance - distance;
      }
      break;
  }
  if (flipMainAxis)
    childMainPosition -= _getMainSize(child);
  switch (_direction) {
    case Axis.horizontal:
      childParentData.offset = new Offset(childMainPosition, childCrossPosition);
      break;
    case Axis.vertical:
      childParentData.offset = new Offset(childCrossPosition, childMainPosition);
      break;
  }
  if (flipMainAxis) {
    childMainPosition -= betweenSpace;
  } else {
    childMainPosition += _getMainSize(child) + betweenSpace;
  }
  child = childParentData.nextSibling;
}

```

最后，则是根据交叉轴的对齐方式设置，对 child 进行位置调整，到此，布局结束。

我们可以顺一下整体的流程：

  - 1.计算出 flex 的总和，并找到最后一个设置了 flex 的 child ；
  - 2.对不包含 flex 的 child ，根据交叉轴对齐方式，对齐进行调整，并计算出主轴方向上所占区域大小；
  - 3.计算出每一份 flex 所占用的空间，并根据交叉轴对齐方式，对包含 flex 的 child 进行调整；
  - 4.如果交叉轴设置为 baseline 对齐，则计算出整体的 baseline 值；
  - 5.按照主轴对齐方式，对 child 进行调整；
  - 6.最后，根据交叉轴对齐方式，对所有 child 位置进行调整，完成布局。


#### Row 使用场景

Row 和 Column 都是非常常用的布局控件。一般情况下，比方说需要将控件在一行或者一列显示的时候，都可以使用。但并不是说只能使用 Row 或者 Column 去布局，也可以使用 Stack ，看具体的场景选择。


### Column

在讲解 Row 的时候，其实是按照 Flex 的一些布局行为来进行的，包括源码分析，也都是在用 Flex 进行分析的。 Row 和  Column 都是 Flex 的子类，只是 direction 参数不同。 Column 各方面同 Row ，因此在这里不再另行讲解。

