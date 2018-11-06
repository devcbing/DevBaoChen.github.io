---
layout: post
title: SizedOverflowBox、Transform、CustomSingleChildLayout详解
categories: [Flutter, ]
description: SizedOverflowBox、Transform、CustomSingleChildLayout详解
keywords: Flutter, 
---

SizedOverflowBox、Transform、CustomSingleChildLayout详解

![](/images/posts/flutter/2018-09-30-00.jpeg)

### SizedOverflowBox

 SizedOverflowBox 是 SizedBox 与 OverflowBox 的结合体

#### SizedOverflowBox 布局

- 1.尺寸部分。通过将自身的固定尺寸，传递给 child ，来达到控制 child 尺寸的目的；

- 2.超出部分。可以突破父节点尺寸的限制，超出部分也可以被渲染显示，与 OverflowBox 类似。

#### SizedOverflowBox 继承关系

```

Object > Diagnosticable > DiagnosticableTree > Widget > RenderObjectWidget > SingleChildRenderObjectWidget > SizedOverflowBox

```

#### SizedOverflowBox 示例

```

Container(
  color: Colors.green,
  alignment: Alignment.topRight,
  width: 200.0,
  height: 200.0,
  padding: EdgeInsets.all(5.0),
  child: SizedOverflowBox(
    size: Size(100.0, 200.0),
    child: Container(color: Colors.red, width: 200.0, height: 100.0,),
  ),
);


```

#### SizedOverflowBox 源码

##### 构造函数

```

const SizedOverflowBox({
  Key key,
  @required this.size,
  this.alignment = Alignment.center,
  Widget child,
})

```

##### 属性解析

`size `: 固定的尺寸；

`alignment `: 对齐方式。


##### 源码

```
size = constraints.constrain(_requestedSize);
if (child != null) {
  child.layout(constraints);
  alignChild();
}

```

如果 child 存在的话，就将 child 设为对应的尺寸，然后按照对齐方式进行对齐


### Transform

Transform 主要是做矩阵变换的。 Container 中矩阵变换就是使用的 Transform 。

#### Transform 布局

对 child 进行平移、旋转、缩放等操作.

#### Transform 继承关系

```

Object > Diagnosticable > DiagnosticableTree > Widget > RenderObjectWidget > SingleChildRenderObjectWidget > Transform

```

#### Transform 示例

```
Center(
  child: Transform(
    transform: Matrix4.rotationZ(0.3),
    child: Container(
      color: Colors.blue,
      width: 100.0,
      height: 100.0,
    ),
  ),
)

```
将 Container 绕 z 轴旋转了

#### Transform 源码

##### 构造函数

默认构造函数

```
const Transform({
  Key key,
  @required this.transform,
  this.origin,
  this.alignment,
  this.transformHitTests = true,
  Widget child,
})
```

其他构造函数

```
Transform.rotate
Transform.translate
Transform.scale

```

##### 属性解析

`transform `：一个 4x4 的矩阵。不难发现，其他平台的变换矩阵也都是四阶的。一些复合操作，仅靠三维是不够的，必须采用额外的一维来补充，感兴趣的同学可以自行搜索了解。

`origin `：旋转点，相对于左上角顶点的偏移。默认旋转点事左上角顶点。

`alignment `：对齐方式。

`transformHitTests `：点击区域是否也做相应的改变。


##### 源码

```
if (child != null) {
  final Matrix4 transform = _effectiveTransform;
  final Offset childOffset = MatrixUtils.getAsTranslation(transform);
  if (childOffset == null)
    context.pushTransform(needsCompositing, offset, transform, super.paint);
  else
    super.paint(context, offset + childOffset);
}

```

整个绘制代码不复杂，如果 child 有偏移的话，则将两个偏移相加，进行绘制。如果 child 没有偏移的话，则按照设置的 offset 、 transform 进行绘制。


#### Transform 使用场景

在平移、旋转、缩放等功能中需要用到。而且只是单纯的进行变换的话，用 Transform 比用 Container 效率会更高。


### CustomSingleChildLayout

通过外部传入的布局行为，来进行布局的控件，不同于其他固定布局的控件，我们自定义一些单节点布局控件的时候，可以考虑使用它。

#### CustomSingleChildLayout 布局

 CustomSingleChildLayout 提供了一个控制 child 布局的 delegate ，这个 delegate 可以控制这些因素：

- 1.可以控制 child 的布局 constraints ；
- 2.可以控制 child 的位置；
- 3.在 parent 的尺寸不依赖于 child 的情况下，可以决定 parent 的尺寸。

#### CustomSingleChildLayout 继承关系

```
Object > Diagnosticable > DiagnosticableTree > Widget > RenderObjectWidget > SingleChildRenderObjectWidget > CustomSingleChildLayout

```

#### CustomSingleChildLayout 示例

```

class FixedSizeLayoutDelegate extends SingleChildLayoutDelegate {
  FixedSizeLayoutDelegate(this.size);

  final Size size;

  @override
  Size getSize(BoxConstraints constraints) => size;

  @override
  BoxConstraints getConstraintsForChild(BoxConstraints constraints) {
    return new BoxConstraints.tight(size);
  }

  @override
  bool shouldRelayout(FixedSizeLayoutDelegate oldDelegate) {
    return size != oldDelegate.size;
  }
}

Container(
  color: Colors.blue,
  padding: const EdgeInsets.all(5.0),
  child: CustomSingleChildLayout(
    delegate: FixedSizeLayoutDelegate(Size(200.0, 200.0)),
    child: Container(
      color: Colors.red,
      width: 100.0,
      height: 300.0,
    ),
  ),
)

```

由于 SingleChildLayoutDelegate 是一个抽象类，我们需要单独写一个继承自它的 delegate ，来进行相关的布局操作。上面例子中是一个固定尺寸的布局。

#### CustomSingleChildLayout 源码

##### 构造函数

```
const CustomSingleChildLayout({
  Key key,
  @required this.delegate,
  Widget child
})

```

##### 属性解析

`alignment ` ：一个抽象类，我们需要自行实现布局的类。

##### 源码

布局函数：

```
size = _getSize(constraints);
if (child != null) {
  final BoxConstraints childConstraints = delegate.getConstraintsForChild(constraints);
  assert(childConstraints.debugAssertIsValid(isAppliedConstraint: true));
  child.layout(childConstraints, parentUsesSize: !childConstraints.isTight);
  final BoxParentData childParentData = child.parentData;
  childParentData.offset = delegate.getPositionForChild(size, childConstraints.isTight ? childConstraints.smallest : child.size);
}

```

其 child 的 constraints 是通过 delegate 传入的，而这个 delegate 则是我们通过外部继承自  SingleChildLayoutDelegate 实现的。


SingleChildLayoutDelegate 提供的接口：

```

Size getSize(BoxConstraints constraints) => constraints.biggest;
BoxConstraints getConstraintsForChild(BoxConstraints constraints) => constraints;
Offset getPositionForChild(Size size, Size childSize) => Offset.zero;
bool shouldRelayout(covariant SingleChildLayoutDelegate oldDelegate);

```

其中前三个都是布局相关的，包括尺寸、 constraints 、位置。最后一个函数是判断是否需要重新布局的。我们在编写 delegate  的时候，可以选择需要进行修改的属性进行重写，并给予相应的布局属性即可。

#### CustomSingleChildLayout 使用场景

通过这个控件去封装一些基础的控件 ,方便使用。

