---
layout: post
title: Baseline 、FractionallySizedBox 、IntrinsicHeight 、IntrinsicWidth 详解
categories: [Flutter, ]
description: Baseline 、FractionallySizedBox 、IntrinsicHeight 、IntrinsicWidth 详解
keywords: Flutter, 
---

Baseline 、FractionallySizedBox 、IntrinsicHeight 、IntrinsicWidth 详解

![](/images/posts/flutter/2018-09-30-00.jpeg)

### Baseline

Baseline 一般在文字排版的时候会用到，它的作用主要是根据 child 的 baseline 来调整 child 的位置。

#### Baseline 布局

- 1.如果 child 有 baseline ，则根据 child 的 baseline 属性，调整 child 的位置；
- 2.如果 child 没有 baseline ，则根据 child 的 bottom ，来调整 child 的位置。

#### Baseline 继承关系

```

Object > Diagnosticable > DiagnosticableTree > Widget > RenderObjectWidget > SingleChildRenderObjectWidget > Baseline

```

#### Baseline 示例

```

new Row(
  mainAxisAlignment: MainAxisAlignment.spaceBetween,
  children: <Widget>[
    new Baseline(
      baseline: 50.0,
      baselineType: TextBaseline.alphabetic,
      child: new Text(
        'TjTjTj',
        style: new TextStyle(
          fontSize: 20.0,
          textBaseline: TextBaseline.alphabetic,
        ),
      ),
    ),
    new Baseline(
      baseline: 50.0,
      baselineType: TextBaseline.alphabetic,
      child: new Container(
        width: 30.0,
        height: 30.0,
        color: Colors.red,
      ),
    ),
    new Baseline(
      baseline: 50.0,
      baselineType: TextBaseline.alphabetic,
      child: new Text(
        'RyRyRy',
        style: new TextStyle(
          fontSize: 35.0,
          textBaseline: TextBaseline.alphabetic,
        ),
      ),
    ),
  ],
)

```

运行结果是左右两个文本跟中间的 Container 底部在一个水平线上。

#### Baseline 源码

##### 构造函数
```

const Baseline({
  Key key,
  @required this.baseline,
  @required this.baselineType,
  Widget child
})

```
##### 属性解析

`baseline ` ： baseline 数值，必须要有，从顶部算。

`baselineType ` ：bseline 类型，也是必须要有的，目前有两种类型

- 1. alphabetic ：对齐字符底部的水平线；
- 2. ideographic ：对齐表意字符的水平线。


##### 源码

```

child.layout(constraints.loosen(), parentUsesSize: true);
final double childBaseline = child.getDistanceToBaseline(baselineType);
final double actualBaseline = baseline;
final double top = actualBaseline - childBaseline;
final BoxParentData childParentData = child.parentData;
childParentData.offset = new Offset(0.0, top);
final Size childSize = child.size;
size = constraints.constrain(new Size(childSize.width, top + childSize.height));

```

getDistanceToBaseline 这个函数是获取 baseline 数值的，存在的话，就取这个值，不存在的话，则取其高度。

计算过程：

- 1.获取 child 的 baseline 值；
- 2.计算出 top 值，其为 baseline - childBaseline ，这个值有可能为负数；
- 3.计算出 Baseline 控件尺寸，宽度为 child 的，高度则为 top + childSize.height 。

#### Baseline 使用场景

 主要使用在字符串对齐需求中。


### FractionallySizedBox

FractionallySizedBox  会根据现有的空间来调整 child 的尺寸， 所以 child 设置了具体数值也是没有作用的。

#### FractionallySizedBox 布局

FractionallySizedBox 的布局主要跟他的宽高因子有关，当参数为 null ，或具有具体数值的时候，布局是不一样的，  FractionallySizedBox  还有一个 alignment 属性，来进行布局对齐;

- 1.当设置了具体的宽高后， 具体宽高则根据空间宽高 * 宽高因子，当 `宽高因子` 大于 1 时，宽高会大于父控件的范围。
- 2.当没有设置具体的宽高时，则会填满具体可用范围。

#### FractionallySizedBox 继承关系

```

Object > Diagnosticable > DiagnosticableTree > Widget > RenderObjectWidget > SingleChildRenderObjectWidget > FractionallySizedBox


```
#### FractionallySizedBox 示例

```
new Container(
    color: Colors.blue,
    height: 150.0,
    width: 150.0,
    padding: const EdgeInsets.all(10.0),
    child: new FractionallySizedBox(
      alignment: Alignment.topLeft,
      widthFactor: 1.5,
      heightFactor: 0.5,
      child: new Container(
        color: Colors.red,
      ),
    ),
  )
```

![](/images/posts/flutter/2018-10-06-01.png)

#### FractionallySizedBox 源码

##### 构造函数

```
const FractionallySizedBox({
  Key key,
  this.alignment = Alignment.center,
  this.widthFactor,
  this.heightFactor,
  Widget child,
})

```

##### 属性解析

`alignment ` ：对齐方式，不能为 null 。

`widthFactor ` ：宽度因子，跟之前介绍的控件类似，宽度乘以这个值，就是最后的宽度。

`heightFactor `：高度因子，用作计算最后实际高度的。

其中 widthFactor 和 heightFactor 都有一个规则;

- 1.如果不为 null ，那么实际的最大宽高度则为 child 的宽高乘以这个因子；

- 2.如果为 null ，那么 child 的宽高则会尽量充满整个区域。

##### 源码


FractionallySizedBox 内部具体渲染是由 RenderFractionallySizedOverflowBox 来实现的，通过命名就可以看出，这个控件可能会 Overflow
```

double minWidth = constraints.minWidth;
double maxWidth = constraints.maxWidth;
if (_widthFactor != null) {
  final double width = maxWidth * _widthFactor;
  minWidth = width;
  maxWidth = width;
}
double minHeight = constraints.minHeight;
double maxHeight = constraints.maxHeight;
if (_heightFactor != null) {
  final double height = maxHeight * _heightFactor;
  minHeight = height;
  maxHeight = height;
}

```

FractionallySizedBox 根据宽高因子是否存在，来进行相对应的尺寸计算。

#### FractionallySizedBox 使用场景

当需要在一个区域里面取百分比尺寸的时候，可以使用这个，比方说，高度40%宽度70%的区域。当然， AspectRatio 也可以达到近似的效果。


### IntrinsicHeight

IntrinsicHeight 的作用是调整 child 到固定的高度。

#### IntrinsicHeight 布局

IntrinsicHeight 将可能高度不受限制的 child ，调整到一个合适的尺寸。

#### IntrinsicHeight 继承关系

```
Object > Diagnosticable > DiagnosticableTree > Widget > RenderObjectWidget > SingleChildRenderObjectWidget > IntrinsicHeight

```

#### IntrinsicHeight 示例

```

new IntrinsicHeight(
  child: new Row(
    mainAxisAlignment: MainAxisAlignment.spaceBetween,
    children: <Widget>[
      new Container(color: Colors.blue, width: 100.0),
      new Container(color: Colors.red, width: 50.0,height: 50.0,),
      new Container(color: Colors.yellow, width: 150.0),
    ],
  ),
);

```

![](/images/posts/flutter/2018-10-06-02.png)


当没有 IntrinsicHeight 包裹着，可以看到，第一三个 Container 高度是不受限制的，当外层套一个 IntrinsicHeight ，第一三个 Container 高度就调整到第二个一样的高度。

#### IntrinsicHeight 源码

##### 构造函数

```

const IntrinsicHeight({ Key key, Widget child })

```

##### 属性解析

IntrinsicHeight 除了 child ,没有其他属性

##### 源码

当child不为null的时候，

```

BoxConstraints childConstraints = constraints;
if (!childConstraints.hasTightHeight) {
  final double height = child.getMaxIntrinsicHeight(childConstraints.maxWidth);
  assert(height.isFinite);
  childConstraints = childConstraints.tighten(height: height);
}
child.layout(childConstraints, parentUsesSize: true);
size = child.size;

```

首先会检测是否只有一个高度值满足约束条件，如果不是的话，则返回一个最小的高度。然后调整尺寸。



### IntrinsicWidth

IntrinsicWidth 从描述看，跟 IntrinsicHeight 类似，一个是调整高度，一个是调整宽度。同样是会存在效率问题，能别使用就尽量别使用。

#### IntrinsicWidth 布局行为

IntrinsicWidth 不同于 IntrinsicHeight ，它包含了额外的两个参数， stepHeight 以及 stepWidth 。而  IntrinsicWidth 的布局行为跟这两个参数相关

- 1.当 stepWidth 不是 null 的时候， child 的宽度将会是 stepWidth 的倍数，当 stepWidth 值比 child 最小宽度小的时候，这个值不起作用；
- 2.当 stepWidth 为 null 的时候， child 的宽度是 child 的最小宽度；
- 3.当 stepHeight 不为 null 的时候，效果跟 stepWidth 相同；
- 4.当 stepHeight 为 null 的时候，高度取最大高度。


#### IntrinsicWidth 继承关系

```

Diagnosticable > DiagnosticableTree > Widget > RenderObjectWidget > SingleChildRenderObjectWidget > IntrinsicWidth

```

#### IntrinsicWidth 示例

```

new Container(
  color: Colors.green,
  padding: const EdgeInsets.all(5.0),
  child: new IntrinsicWidth(
    stepHeight: 450.0,
    stepWidth: 300.0,
    child: new Column(
      children: <Widget>[
        new Container(color: Colors.blue, height: 100.0),
        new Container(color: Colors.red, width: 150.0, height: 100.0),
        new Container(color: Colors.yellow, height: 150.0,),
      ],
    ),
  ),
)

```

![](/images/posts/flutter/2018-10-06-03.png)

分别对 stepWidth 以及 stepHeight 设置不同的值，可以看到不同的效果，当 step 值比最小宽高小的时候，这个值其实是不起作用的。


#### IntrinsicWidth 源码

##### 构造函数

```
const IntrinsicWidth({ Key key, this.stepWidth, this.stepHeight, Widget child })

```

##### 属性解析

`stepWidth `：可以为 null ，效果参看上面所说的布局行为。
`stepHeight `：可以为 null ，效果参看上面所说的布局行为。

##### 源码

_applyStep 函数

```
static double _applyStep(double input, double step) {
assert(input.isFinite);
if (step == null)
  return input;
return (input / step).ceil() * step;
}

```

如果存在 step 数值的话，则会是 step 的倍数，如果 step 为 null，则返回原始的尺寸。

接下来我们看看 child 不为 null 时候的布局代码

```

BoxConstraints childConstraints = constraints;
if (!childConstraints.hasTightWidth) {
  final double width = child.getMaxIntrinsicWidth(childConstraints.maxHeight);
  assert(width.isFinite);
  childConstraints = childConstraints.tighten(width: _applyStep(width, _stepWidth));
}
if (_stepHeight != null) {
  final double height = child.getMaxIntrinsicHeight(childConstraints.maxWidth);
  assert(height.isFinite);
  childConstraints = childConstraints.tighten(height: _applyStep(height, _stepHeight));
}
child.layout(childConstraints, parentUsesSize: true);
size = child.size;

```

宽度方面的布局跟 IntrinsicHeight 高度部分相似，只是多了一个 step 的额外数值。总体的布局表现跟上面分析的布局行为一致，根据 step 值是否是 null 来进行判断，但是注意其对待高度与宽度的表现略有差异。

