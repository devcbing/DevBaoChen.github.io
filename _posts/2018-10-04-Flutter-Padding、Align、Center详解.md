---
layout: post
title: Padding、Align、Center详解
categories: [Flutter, ]
description: Padding、Align、Center详解
keywords: Flutter, 
---

Padding、Align、Center详解

### Padding

Padding : 设置 widget 的内边距 。

#### Padding 的布局 
- 当 child 为空的时候，会产生一个宽为 left + right ，高为 top + bottom 的区域；
- 当 child 不为空的时候，Padding 会将布局约束传递给 child ，根据设置的 padding 属性，缩小 child 的布局尺寸。然后  Padding 将自己调整到 child 设置了 padding 属性的尺寸，在 child 周围创建空白区域。 

#### Padding 继承关系

```
Object > Diagnosticable > DiagnosticableTree > Widget > RenderObjectWidget > SingleChildRenderObjectWidget > Padding

```
可以看出：Padding 控件是一个基础控件，不像 Container 这种组合控件。 Container 中的 margin 以及 padding 属性都是利用 Padding 控件去实现的。

##### SingleChildRenderObjectWidget

SingleChildRenderObjectWidget 是 RenderObjectWidgets 的一个子类，用于限制只能有一个子节点。它只提供 child 的存储，而不提供实际的更新逻辑

##### RenderObjectWidgets

RenderObjectWidgets 为 RenderObjectElement 提供配置，而 RenderObjectElement 包含着（ wrap ）RenderObject ， RenderObject 则是在应用中提供实际的绘制（ rendering ）的元素。

#### Padding 例子

```
new Padding(
  padding: new EdgeInsets.all(8.0),
  child: const Card(child: const Text('Hello World!')),
)//为 Card 设置一个宽度为 8 的内边距

```

#### Padding 源码
 
 构造函数
```
const Padding({
    Key key,
    @required this.padding,
    Widget child,
  })
```
 构造函数中仅仅包含了一个 padding 属性； padding 的类型为 `EdgeInsetsGeometry `, EdgeInsetsGeometry 是    EdgeInsets 以及 EdgeInsetsDirectional 的基类。在实际使用中不涉及到国际化，例如适配阿拉伯地区等，一般都是使用  EdgeInsets 。EdgeInsetsDirectional 光看命名就知道跟方向相关，因此它的四个边距不限定上下左右，而是根据方向来定的

源码
```
@override
  RenderPadding createRenderObject(BuildContext context) {
    return new RenderPadding(
      padding: padding,
      textDirection: Directionality.of(context),
   );
}

```
Padding 的创建函数，实际上是由 RenderPadding 来进行的;

关于 RenderPadding 的实际布局表现，当 child 为 null 的时候：

```
if (child == null) {
  size = constraints.constrain(new Size(
    _resolvedPadding.left + _resolvedPadding.right,
    _resolvedPadding.top + _resolvedPadding.bottom
  ));
  return;
}

```

返回一个宽为 _resolvedPadding.left + _resolvedPadding.right ，高为 _resolvedPadding.top + _resolvedPadding.bottom 的区域，

当 child 不为 null 的时候，经历了三个过程，即调整 child 尺寸、调整 child 位置以及调整 Padding 尺寸，最终达到实际的布局效果。

```

// 调整child尺寸
final BoxConstraints innerConstraints = constraints.deflate(_resolvedPadding);
child.layout(innerConstraints, parentUsesSize: true);

// 调整child位置
final BoxParentData childParentData = child.parentData;
childParentData.offset = new Offset(_resolvedPadding.left, _resolvedPadding.top);

// 调整Padding尺寸
size = constraints.constrain(new Size(
  _resolvedPadding.left + child.size.width + _resolvedPadding.right,
  _resolvedPadding.top + child.size.height + _resolvedPadding.bottom
));

```

#### 使用场景
Padding 本身还是挺简单的，基本上需要间距的地方，它都能够使用。如果在单一的间距场景，使用 Padding 比 Container 的成本要小一些，毕竟 Container 里面包含了多个 widget 。 Padding 能够实现的， Container 都能够实现，只不过， Container 更加的复杂。


### Align

在使用其他方式进行开发时， Align 都是被当做一个控件的属性，并没有单独成为一个控件。Align 本身实现的功能并不复杂，设置 child 的对齐方式，例如居中、居左居右等，并根据 child 尺寸调节自身尺寸。

#### Align 的布局

- 当 widthFactor 和 heightFactor 为 null 的时候，当其有限制条件的时候， Align 会根据限制条件尽量的扩展自己的尺寸，当没有限制条件的时候，会调整到 child 的尺寸；

- 当 widthFactor 或者 heightFactor 不为 null 的时候，Aligin会根据factor属性，扩展自己的尺寸，例如设置widthFactor为2.0的时候，那么，Align的宽度将会是child的两倍。


Align 这么设置，原因很简单，设置对齐方式的话，如果外层元素尺寸不确定的话，内部的对齐就无法确定。因此，会有宽高因子、根据外层限制扩大到最大尺寸、外层不确定时调整到 child 尺寸这些行为。

#### Align 继承关系

```
Object > Diagnosticable > DiagnosticableTree > Widget > RenderObjectWidget > SingleChildRenderObjectWidget > Align

```
Align 和 Padding 一样，也是一个非常基础的组件，Container 中的 align 属性，也是使用 Align 去实现的。

#### 示例代码

```

new Align(
  alignment: Alignment.center,
  widthFactor: 2.0,
  heightFactor: 2.0,
  child: new Text("Align"),
)//  设置一个宽高为child两倍区域的Align，其child处在正中间。


```

#### 源码

```
const Align({
    Key key,
    this.alignment: Alignment.center,
    this.widthFactor,
    this.heightFactor,
    Widget child
  })

```
Align 的构造函数基本上就是宽高因子、对齐方式属性。日常使用中，宽高因子属性基本上用的不多。如果是复杂的布局， Container 内部的 align 属性也可以实现相同的效果

##### 属性解析

` alignment `：对齐方式，一般会使用系统默认提供的9种方式，但是并不是说只有这9种，例如如下的定义。系统提供的9种方式只是预先定义好的。


```
/// The top left corner.
static const Alignment topLeft = const Alignment(-1.0, -1.0);

```

Alignment 实际上是包含了两个属性的，其中第一个参数，-1.0 是左边对齐， 1.0 是右边对齐，第二个参数，-1.0 是顶部对齐，1.0是底部对齐。根据这个规则，我们也可以自定义我们需要的对齐方式，例如

```
/// 居右高于底部1/4处.
static const Alignment rightHalfBottom = alignment: const Alignment(1.0, 0.5);

```
` widthFactor `：宽度因子，如果设置的话， Align 的宽度就是 child 的宽度乘以这个值，不能为负数。

` heightFactor `：高度因子，如果设置的话， Align 的高度就是 child 的高度乘以这个值，不能为负数。

##### 源码

```

@override
  RenderPositionedBox createRenderObject(BuildContext context) {
    return new RenderPositionedBox(
      alignment: alignment,
      widthFactor: widthFactor,
      heightFactor: heightFactor,
      textDirection: Directionality.of(context),
    );
  }

```

Align 的实际构造调用的是 ` RenderPositionedBox `

RenderPositionedBox 的布局 ：


```

// 根据_widthFactor、_heightFactor以及限制因素来确定宽高
final bool shrinkWrapWidth = _widthFactor != null || constraints.maxWidth == double.infinity;
final bool shrinkWrapHeight = _heightFactor != null || constraints.maxHeight == double.infinity;

if (child != null) {
  //  如果child不为null，则根据规则设置Align的宽高，如果需要缩放，则根据_widthFactor是否为null来进行缩放，如果不需要，则尽量扩展。
  child.layout(constraints.loosen(), parentUsesSize: true);
  size = constraints.constrain(new Size(shrinkWrapWidth ? child.size.width * (_widthFactor ?? 1.0) : double.infinity,
                                        shrinkWrapHeight ? child.size.height * (_heightFactor ?? 1.0) : double.infinity));
  alignChild();
} else {
  // 如果child为null，如果需要缩放，则变为0，否则就尽量扩展
  size = constraints.constrain(new Size(shrinkWrapWidth ? 0.0 : double.infinity,
                                        shrinkWrapHeight ? 0.0 : double.infinity));
}

```

##### Align 使用场景

一般在对齐场景下使用，例如需要右对齐或者底部对齐之类的。它能够实现的功能， Container 都能实现。

### Center

 Center 继承自 Align ，只不过是将 alignment 设置为 Alignment.center ，其他属性例如 widthFactor 、  heightFactor ，布局行为，都与 Align 完全一样，在这里就不再单独做介绍了。 Center 源码如下，没有设置 alignment 属性，是因为 Align 默认的对齐方式就是居中。

```

    class Center extends Align {
    /// Creates a widget that centers its child.
    const Center({ Key key, double widthFactor, double heightFactor, Widget child })
        : super(key: key, widthFactor: widthFactor, heightFactor: heightFactor, child: child);
    }

```