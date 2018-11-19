---
layout: post
title: Flow、Table、Wrap详解
categories: [Flutter, ]
description: Flow、Table、Wrap详解
keywords: Flutter, 
---

Flow、Table、Wrap详解


![](/images/posts/flutter/2018-09-30-00.jpeg)


### Flow

Flow 是一个实现流式布局算法的控件。流式布局在大前端是很常见的布局方式，但是一般使用 Flow 很少，因为其过于复杂，很多场景下都会去使用 Wrap 。

#### Flow 布局行为

 Flow 官方介绍是一个对 child 尺寸以及位置调整非常高效的控件，主要是得益于其 FlowDelegate 。另外 Flow 在用转换矩阵（ transformation matrices ）对 child 进行位置调整的时候进行了优化。

 Flow 以及其 child 的一些约束都会受到 FlowDelegate 的控制，例如重写 FlowDelegate 中的 getSize ，可以设置 Flow  的尺寸，重写其 getConstraintsForChild 方法，可以设置每个 child 的布局约束条件。

Flow 之所以高效，是因为其在定位过后，如果使用 FlowDelegate 中的 paintChildren 改变 child 的尺寸或者位置，只是重绘，并没有实际调整其位置。


#### Flow 继承关系

```
Object > Diagnosticable > DiagnosticableTree > Widget > RenderObjectWidget > MultiChildRenderObjectWidget > Flow

```

#### Flow 示例

```

const width = 80.0;
const height = 60.0;

Flow(
  delegate: TestFlowDelegate(margin: EdgeInsets.fromLTRB(10.0, 10.0, 10.0, 10.0)),
  children: <Widget>[
    new Container(width: width, height: height, color: Colors.yellow,),
    new Container(width: width, height: height, color: Colors.green,),
    new Container(width: width, height: height, color: Colors.red,),
    new Container(width: width, height: height, color: Colors.black,),
    new Container(width: width, height: height, color: Colors.blue,),
    new Container(width: width, height: height, color: Colors.lightGreenAccent,),
  ],
)

class TestFlowDelegate extends FlowDelegate {
  EdgeInsets margin = EdgeInsets.zero;

  TestFlowDelegate({this.margin});
  @override
  void paintChildren(FlowPaintingContext context) {
    var x = margin.left;
    var y = margin.top;
    for (int i = 0; i < context.childCount; i++) {
      var w = context.getChildSize(i).width + x + margin.right;
      if (w < context.size.width) {
        context.paintChild(i,
            transform: new Matrix4.translationValues(
                x, y, 0.0));
        x = w + margin.left;
      } else {
        x = margin.left;
        y += context.getChildSize(i).height + margin.top + margin.bottom;
        context.paintChild(i,
            transform: new Matrix4.translationValues(
                x, y, 0.0));
        x += context.getChildSize(i).width + margin.left + margin.right;
      }
    }
  }

  @override
  bool shouldRepaint(FlowDelegate oldDelegate) {
    return oldDelegate != this;
  }
}

```

样例其实并不复杂， FlowDelegate 需要自己实现 child 的绘制，其实大多数时候就是位置的摆放。上面例子中，对每个 child 按照给定的 margin 值，进行排列，如果超出一行，则在下一行进行布局。

![](/images/posts/flutter/2018-10-11-01.png)


上述 child 宽度的变化，这个例子是没问题的，如果每个 child 的高度不同，则需要对代码进行调整，具体的调整是换行的时候，需要根据上一行的最大高度来确定下一行的起始 y 坐标。

#### Flow 源码

##### 构造函数

```
Flow({
  Key key,
  @required this.delegate,
  List<Widget> children = const <Widget>[],
})

```

##### 属性解析

`delegate `：影响 Flow 具体布局的 FlowDelegate 。

其中 FlowDelegate 包含如下几个方法：

 - 1.getConstraintsForChild : 设置每个 child 的布局约束条件，会覆盖已有的；

 - 2.getSize：设置 Flow 的尺寸；

 - 3.paintChildren： child 的绘制控制代码，可以调整尺寸位置，写起来比较的繁琐；

 - 4.shouldRepaint：是否需要重绘；

 - 5.shouldRelayout：是否需要重新布局。

一般会使用到 paintChildren 以及 shouldRepaint 两个方法。


##### 源码

 Flow 的布局代码

```

Size _getSize(BoxConstraints constraints) {
  assert(constraints.debugAssertIsValid());
  return constraints.constrain(_delegate.getSize(constraints));
}

@override
void performLayout() {
  size = _getSize(constraints);
  int i = 0;
  _randomAccessChildren.clear();
  RenderBox child = firstChild;
  while (child != null) {
    _randomAccessChildren.add(child);
    final BoxConstraints innerConstraints = _delegate.getConstraintsForChild(i, constraints);
    child.layout(innerConstraints, parentUsesSize: true);
    final FlowParentData childParentData = child.parentData;
    childParentData.offset = Offset.zero;
    child = childParentData.nextSibling;
    i += 1;
  }
}

```

可以看到 Flow 尺寸的取值，直接来自于 delegate 的 getSize 方法。对于每一个 child ，则是将 delegate 中的  getConstraintsForChild 设置的约束条件，设置在 child 上。

Flow 布局上的表现，受 Delegate 中 getSize 以及 getConstraintsForChild 两个方法的影响。第一个方法设置其尺寸，第二个方法设置其 children 的布局约束条件。


Flow 绘制方法。

```

void _paintWithDelegate(PaintingContext context, Offset offset) {
  _lastPaintOrder.clear();
  _paintingContext = context;
  _paintingOffset = offset;
  for (RenderBox child in _randomAccessChildren) {
    final FlowParentData childParentData = child.parentData;
    childParentData._transform = null;
  }
  try {
    _delegate.paintChildren(this);
  } finally {
    _paintingContext = null;
    _paintingOffset = null;
  }
}


```

先将上次设置的参数都初始化，然后调用 delegate 中的 paintChildren 进行绘制。在 paintChildren 中会调用 paintChild  方法去绘制每个 child ，我们接下来看下其代码。

```

@override
  void paintChild(int i, { Matrix4 transform, double opacity = 1.0 }) {
    transform ??= new Matrix4.identity();
    final RenderBox child = _randomAccessChildren[i];
    final FlowParentData childParentData = child.parentData;
    _lastPaintOrder.add(i);
    childParentData._transform = transform;
    
    if (opacity == 0.0)
      return;

    void painter(PaintingContext context, Offset offset) {
      context.paintChild(child, offset);
    }
    
    if (opacity == 1.0) {
      _paintingContext.pushTransform(needsCompositing, _paintingOffset, transform, painter);
    } else {
      _paintingContext.pushOpacity(_paintingOffset, _getAlphaFromOpacity(opacity), (PaintingContext context, Offset offset) {
        context.pushTransform(needsCompositing, offset, transform, painter);
      });
    }
  }

```

paitChild 函数首先会将 transform 值设在 child 上，然后根据 opacity 值，决定其绘制的表现。

 - 1.当 opacity 为 0 时，只是设置了 transform 值，这样做是为了让其响应区域跟随调整，虽然不显示出来；

 - 2.当 opacity 为 1 的时候，只是进行 Transform 操作；

 - 3.当 opacity 大于 0 小于1时，先调整其透明度，再进行 Transform 操作。


至于其为什么高效，主要是因为它的布局函数不牵涉到 child 的布局，而在绘制的时候，则根据 delegate 中的策略，进行有效的绘制。

#### Flow 使用场景

Flow 在一些定制化的流式布局中，有可用场景，但是一般写起来比较复杂，但胜在灵活性以及其高效。



### Table

实现表格布局

#### Table 布局

表格的每一行的高度，由其内容决定，每一列的宽度，则由 columnWidths 属性单独控制。


#### Table 继承关系

```

Object > Diagnosticable > DiagnosticableTree > Widget > RenderObjectWidget > Table

```

#### Table 示例

```
Table(
  columnWidths: const <int, TableColumnWidth>{
    0: FixedColumnWidth(50.0),
    1: FixedColumnWidth(100.0),
    2: FixedColumnWidth(50.0),
    3: FixedColumnWidth(100.0),
  },
  border: TableBorder.all(color: Colors.red, width: 1.0, style: BorderStyle.solid),
  children: const <TableRow>[
    TableRow(
      children: <Widget>[
        Text('A1'),
        Text('B1'),
        Text('C1'),
        Text('D1'),
      ],
    ),
    TableRow(
      children: <Widget>[
        Text('A2'),
        Text('B2'),
        Text('C2'),
        Text('D2'),
      ],
    ),
    TableRow(
      children: <Widget>[
        Text('A3'),
        Text('B3'),
        Text('C3'),
        Text('D3'),
      ],
    ),
  ],
)

```

一个三行四列的表格，第一三行宽度为50，第二四行宽度为100。

#### Table 源码

##### 构造函数

```

Table({
  Key key,
  this.children = const <TableRow>[],
  this.columnWidths,
  this.defaultColumnWidth = const FlexColumnWidth(1.0),
  this.textDirection,
  this.border,
  this.defaultVerticalAlignment = TableCellVerticalAlignment.top,
  this.textBaseline,
})

```

##### 属性解析

`columnWidths `：设置每一列的宽度。

`defaultColumnWidth `：默认的每一列宽度值，默认情况下均分。

`textDirection `：文字方向，一般无需考虑。

`border `：表格边框。

`defaultVerticalAlignment `：每一个 cell 的垂直方向的 alignment 。

总共包含 5 种：

  - 1.top ：被放置在的顶部；
  - 2.middle ：垂直居中；
  - 3.bottom ：放置在底部；
  - 4.baseline ：文本 baseline 对齐；
  - 5.fill ：充满整个 cell 。

`textBaseline `： defaultVerticalAlignment 为 baseline 的时候，会用到这个属性。

##### 源码

第一步，当行或者列为0的时候，将自身尺寸设为 0x0 。


```

if (rows * columns == 0) {
  size = constraints.constrain(const Size(0.0, 0.0));
  return;
}

```

第二步，根据 textDirection 值，设置方向，一般在阿拉伯语系中，一些文本都是从右往左现实的，平时使用时，不需要去考虑这个属性。

```

switch (textDirection) {
  case TextDirection.rtl:
    positions[columns - 1] = 0.0;
    for (int x = columns - 2; x >= 0; x -= 1)
      positions[x] = positions[x+1] + widths[x+1];
    _columnLefts = positions.reversed;
    tableWidth = positions.first + widths.first;
    break;
  case TextDirection.ltr:
    positions[0] = 0.0;
    for (int x = 1; x < columns; x += 1)
      positions[x] = positions[x-1] + widths[x-1];
    _columnLefts = positions;
    tableWidth = positions.last + widths.last;
    break;
}

```

第三步，设置每一个 cell 的尺寸。

```

  for (int x = 0; x < columns; x += 1) {
    final int xy = x + y * columns;
    final RenderBox child = _children[xy];
    if (child != null) {
      final TableCellParentData childParentData = child.parentData;
      childParentData.x = x;
      childParentData.y = y;
      switch (childParentData.verticalAlignment ?? defaultVerticalAlignment) {
        case TableCellVerticalAlignment.baseline:
          child.layout(new BoxConstraints.tightFor(width: widths[x]), parentUsesSize: true);
          final double childBaseline = child.getDistanceToBaseline(textBaseline, onlyReal: true);
          if (childBaseline != null) {
            beforeBaselineDistance = math.max(beforeBaselineDistance, childBaseline);
            afterBaselineDistance = math.max(afterBaselineDistance, child.size.height - childBaseline);
            baselines[x] = childBaseline;
            haveBaseline = true;
          } else {
            rowHeight = math.max(rowHeight, child.size.height);
            childParentData.offset = new Offset(positions[x], rowTop);
          }
          break;
        case TableCellVerticalAlignment.top:
        case TableCellVerticalAlignment.middle:
        case TableCellVerticalAlignment.bottom:
          child.layout(new BoxConstraints.tightFor(width: widths[x]), parentUsesSize: true);
          rowHeight = math.max(rowHeight, child.size.height);
          break;
        case TableCellVerticalAlignment.fill:
          break;
      }
    }
  }

```

第四步，如果有 baseline 则进行相关设置。

```
if (haveBaseline) {
  if (y == 0)
    _baselineDistance = beforeBaselineDistance;
    rowHeight = math.max(rowHeight, beforeBaselineDistance + afterBaselineDistance);
}

```

第五步，根据 alignment ，调整 child 的位置。

```

for (int x = 0; x < columns; x += 1) {
    final int xy = x + y * columns;
    final RenderBox child = _children[xy];
    if (child != null) {
      final TableCellParentData childParentData = child.parentData;
      switch (childParentData.verticalAlignment ?? defaultVerticalAlignment) {
        case TableCellVerticalAlignment.baseline:
          if (baselines[x] != null)
            childParentData.offset = new Offset(positions[x], rowTop + beforeBaselineDistance - baselines[x]);
          break;
        case TableCellVerticalAlignment.top:
          childParentData.offset = new Offset(positions[x], rowTop);
          break;
        case TableCellVerticalAlignment.middle:
          childParentData.offset = new Offset(positions[x], rowTop + (rowHeight - child.size.height) / 2.0);
          break;
        case TableCellVerticalAlignment.bottom:
          childParentData.offset = new Offset(positions[x], rowTop + rowHeight - child.size.height);
          break;
        case TableCellVerticalAlignment.fill:
          child.layout(new BoxConstraints.tightFor(width: widths[x], height: rowHeight));
          childParentData.offset = new Offset(positions[x], rowTop);
          break;
      }
    }
  }

```

最后一步，则是根据每一行的宽度以及每一列的高度，设置 Table 的尺寸。

```
size = constraints.constrain(new Size(tableWidth, rowTop));

```

最后梳理一下整个的布局流程：

 - 1.当行或者列为0的时候，将自身尺寸设为0x0；
 - 2.根据 textDirection 进行相关设置；
 - 3.设置 cell 的尺寸；
 - 4.如果设置了 baseline ，则进行相关设置；
 - 5.根据 alignment 设置 cell 垂直方向的位置；
 - 6.设置 Table 的尺寸。


#### Table 使用场景

在一些需要表格的需求中，可能用到 Table



### Wrap

其实 Wrap 实现的效果， Flow 可以很轻松，而且可以更加灵活的实现出来。

#### Wrap 布局行为

Flow 可以很轻易的实现 Wrap 的效果，但是 Wrap 更多的是在使用了 Flex 中的一些概念，某种意义上说是跟 Row 、 Column 更加相似的。

单行的 Wrap 跟 Row 表现几乎一致，单列的 Wrap 则跟 Row 表现几乎一致。但 Row 与 Column 都是单行单列的， Wrap 则突破了这个限制，mainAxis 上空间不足时，则向 crossAxis 上去扩展显示。

从效率上讲， Flow 肯定会比 Wrap 高，但是 Wrap 使用起来会方便一些。


#### Wrap 继承关系

```

Object > Diagnosticable > DiagnosticableTree > Widget > RenderObjectWidget > MultiChildRenderObjectWidget > Wrap


```

从继承关系上看， Wrap 与 Flow 都是继承自 MultiChildRenderObjectWidget ， Flow 可以实现 Wrap 的效果，但是两者却是单独实现的，说明两者有很大的不同。

#### Wrap 示例

```

Wrap(
  spacing: 8.0, // gap between adjacent chips
  runSpacing: 4.0, // gap between lines
  children: <Widget>[
    Chip(
      avatar: CircleAvatar(
          backgroundColor: Colors.blue.shade900, child: new Text('AH', style: TextStyle(fontSize: 10.0),)),
      label: Text('Hamilton'),
    ),
    Chip(
      avatar: CircleAvatar(
          backgroundColor: Colors.blue.shade900, child: new Text('ML', style: TextStyle(fontSize: 10.0),)),
      label: Text('Lafayette'),
    ),
    Chip(
      avatar: CircleAvatar(
          backgroundColor: Colors.blue.shade900, child: new Text('HM', style: TextStyle(fontSize: 10.0),)),
      label: Text('Mulligan'),
    ),
    Chip(
      avatar: CircleAvatar(
          backgroundColor: Colors.blue.shade900, child: new Text('JL', style: TextStyle(fontSize: 10.0),)),
      label: Text('Laurens'),
    ),
  ],
)

```

#### Wrap 源码

##### 构造函数

```

Wrap({
  Key key,
  this.direction = Axis.horizontal,
  this.alignment = WrapAlignment.start,
  this.spacing = 0.0,
  this.runAlignment = WrapAlignment.start,
  this.runSpacing = 0.0,
  this.crossAxisAlignment = WrapCrossAlignment.start,
  this.textDirection,
  this.verticalDirection = VerticalDirection.down,
  List<Widget> children = const <Widget>[],
})

```

##### 属性解析

`direction `：主轴（ mainAxis ）的方向，默认为水平。

`alignment `：主轴方向上的对齐方式，默认为 start 。

`spacing `：主轴方向上的间距。

`runAlignment `： run 的对齐方式。 run 可以理解为新的行或者列，如果是水平方向布局的话， run 可以理解为新的一行。

`runSpacing `： run 的间距。

`crossAxisAlignment `：交叉轴（ crossAxis ）方向上的对齐方式。

`textDirection `：文本方向。

`verticalDirection `：定义了 children 摆放顺序，默认是 down ，见 Flex 相关属性介绍。

##### 源码

第一步，如果第一个 child 为 null ，则将其设置为最小尺寸。

```

RenderBox child = firstChild;
if (child == null) {
  size = constraints.smallest;
  return;
}

```

第二步，根据 direction 、 textDirection 以及 verticalDirection 属性，计算出相关的 mainAxis 、 crossAxis 是否需要调整方向，以及主轴方向上的限制。

```

double mainAxisLimit = 0.0;
bool flipMainAxis = false;
bool flipCrossAxis = false;
switch (direction) {
  case Axis.horizontal:
    childConstraints = new BoxConstraints(maxWidth: constraints.maxWidth);
    mainAxisLimit = constraints.maxWidth;
    if (textDirection == TextDirection.rtl)
      flipMainAxis = true;
    if (verticalDirection == VerticalDirection.up)
      flipCrossAxis = true;
    break;
  case Axis.vertical:
    childConstraints = new BoxConstraints(maxHeight: constraints.maxHeight);
    mainAxisLimit = constraints.maxHeight;
    if (verticalDirection == VerticalDirection.up)
      flipMainAxis = true;
    if (textDirection == TextDirection.rtl)
      flipCrossAxis = true;
    break;
}


```

第三步，计算出主轴以及交叉轴的区域大小。

```

while (child != null) {
  child.layout(childConstraints, parentUsesSize: true);
  final double childMainAxisExtent = _getMainAxisExtent(child);
  final double childCrossAxisExtent = _getCrossAxisExtent(child);
  if (childCount > 0 && runMainAxisExtent + spacing + childMainAxisExtent > mainAxisLimit) {
    mainAxisExtent = math.max(mainAxisExtent, runMainAxisExtent);
    crossAxisExtent += runCrossAxisExtent;
    if (runMetrics.isNotEmpty)
      crossAxisExtent += runSpacing;
    runMetrics.add(new _RunMetrics(runMainAxisExtent, runCrossAxisExtent, childCount));
    runMainAxisExtent = 0.0;
    runCrossAxisExtent = 0.0;
    childCount = 0;
  }
  runMainAxisExtent += childMainAxisExtent;
  if (childCount > 0)
    runMainAxisExtent += spacing;
  runCrossAxisExtent = math.max(runCrossAxisExtent, childCrossAxisExtent);
  childCount += 1;
  final WrapParentData childParentData = child.parentData;
  childParentData._runIndex = runMetrics.length;
  child = childParentData.nextSibling;
}

```

第四步，根据 direction 设置 Wrap 的尺寸。

```

switch (direction) {
  case Axis.horizontal:
    size = constraints.constrain(new Size(mainAxisExtent, crossAxisExtent));
    containerMainAxisExtent = size.width;
    containerCrossAxisExtent = size.height;
    break;
  case Axis.vertical:
    size = constraints.constrain(new Size(crossAxisExtent, mainAxisExtent));
    containerMainAxisExtent = size.height;
    containerCrossAxisExtent = size.width;
    break;
}

```

第五步，根据 runAlignment 计算出每一个 run 之间的距离，几种属性的差异，之前文章介绍过，在此就不做详细阐述。

```

final double crossAxisFreeSpace = math.max(0.0, containerCrossAxisExtent - crossAxisExtent);
double runLeadingSpace = 0.0;
double runBetweenSpace = 0.0;
switch (runAlignment) {
  case WrapAlignment.start:
    break;
  case WrapAlignment.end:
    runLeadingSpace = crossAxisFreeSpace;
    break;
  case WrapAlignment.center:
    runLeadingSpace = crossAxisFreeSpace / 2.0;
    break;
  case WrapAlignment.spaceBetween:
    runBetweenSpace = runCount > 1 ? crossAxisFreeSpace / (runCount - 1) : 0.0;
    break;
  case WrapAlignment.spaceAround:
    runBetweenSpace = crossAxisFreeSpace / runCount;
    runLeadingSpace = runBetweenSpace / 2.0;
    break;
  case WrapAlignment.spaceEvenly:
    runBetweenSpace = crossAxisFreeSpace / (runCount + 1);
    runLeadingSpace = runBetweenSpace;
    break;
}

```

第六步，根据 alignment 计算出每一个 run 中 child 的主轴方向上的间距。

```

  switch (alignment) {
    case WrapAlignment.start:
      break;
    case WrapAlignment.end:
      childLeadingSpace = mainAxisFreeSpace;
      break;
    case WrapAlignment.center:
      childLeadingSpace = mainAxisFreeSpace / 2.0;
      break;
    case WrapAlignment.spaceBetween:
      childBetweenSpace = childCount > 1 ? mainAxisFreeSpace / (childCount - 1) : 0.0;
      break;
    case WrapAlignment.spaceAround:
      childBetweenSpace = mainAxisFreeSpace / childCount;
      childLeadingSpace = childBetweenSpace / 2.0;
      break;
    case WrapAlignment.spaceEvenly:
      childBetweenSpace = mainAxisFreeSpace / (childCount + 1);
      childLeadingSpace = childBetweenSpace;
      break;
  }


```

最后一步，调整 child 的位置。

```

  while (child != null) {
    final WrapParentData childParentData = child.parentData;
    if (childParentData._runIndex != i)
      break;
    final double childMainAxisExtent = _getMainAxisExtent(child);
    final double childCrossAxisExtent = _getCrossAxisExtent(child);
    final double childCrossAxisOffset = _getChildCrossAxisOffset(flipCrossAxis, runCrossAxisExtent, childCrossAxisExtent);
    if (flipMainAxis)
      childMainPosition -= childMainAxisExtent;
    childParentData.offset = _getOffset(childMainPosition, crossAxisOffset + childCrossAxisOffset);
    if (flipMainAxis)
      childMainPosition -= childBetweenSpace;
    else
      childMainPosition += childMainAxisExtent + childBetweenSpace;
    child = childParentData.nextSibling;
  }

  if (flipCrossAxis)
    crossAxisOffset -= runBetweenSpace;
  else
    crossAxisOffset += runCrossAxisExtent + runBetweenSpace;


```

我们大致梳理一下布局的流程。

  - 1.如果第一个 child 为 null ，则将 Wrap 设置为最小尺寸，布局结束；
  - 2.根据 direction 、 textDirection 以及 verticalDirection 属性，计算出 mainAxis 、 crossAxis 是否需要调整方向；
  - 3.计算出主轴以及交叉轴的区域大小；
  - 4.根据 direction 设置 Wrap 的尺寸；
  - 5.根据 runAlignment 计算出每一个 run 之间的距离；
  - 6.根据 alignment 计算出每一个 run 中 child 的主轴方向上的间距
  - 7.调整每一个 child 的位置。


#### Wrap 使用场景

对于一些需要按宽度或者高度，让 child 自动换行布局的场景，可以使用，但是 Wrap 可以满足的场景， Flow 一定可以实现，只不过会复杂很多，但是相对的会灵活以及高效很多。
