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


