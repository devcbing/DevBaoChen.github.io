---
layout: post
title: FittedBox、AspectRatio、ConstrainedBox详解
categories: [Flutter, ]
description: FittedBox、AspectRatio、ConstrainedBox详解
keywords: Flutter, 
---

FittedBox 、AspectRatio 、ConstrainedBox 详解


![](/images/posts/flutter/2018-09-30-00.jpeg)


### FittedBox

在 Fluuter 中， FittedBox 的主要作用是进行缩放 （Scale） 和位置调整 (Position);

FittedBox 会在自身的尺寸范围内进行缩放、调整 child 的位置，使得 child 有一个合适的位置。

#### FittedBox 布局

由于 FittedBox 是一个容器，而且主要功能是让起 child 在起范围内进行缩放，所以会存在下面的两种布局情况：

- 1.如果存在外部约束，则按照外部约束对自身尺寸进行调整，然后缩放 child
- 2.如果不存在外部约束，则尺寸跟 child 一样，并且指定的位置和缩放不会起作用 

####  FittedBox 继承关系

```

Object > Diagnosticable > DiagnosticableTree > Widget > RenderObjectWidget > SingleChildRenderObjectWidget > FittedBox

```

所以 FittedBox 是一个基础控件

#### FittedBox 示例

```

new Container(
  color: Colors.amberAccent,
  width: 300.0,
  height: 300.0,
  child: new FittedBox(
    fit: BoxFit.contain,
    alignment: Alignment.topLeft,
    child: new Container(
      color: Colors.red,
      child: new Text("FittedBox"),
    ),
  ),
)

```

#### FittedBox 源码

```
const FittedBox({
Key key,
this.fit: BoxFit.contain,
this.alignment: Alignment.center,
Widget child,
})

```


##### 属性解析

**fit:**
fit：缩放的方式，默认的属性是 BoxFit.contain ， child 在 FittedBox 范围内，尽可能的大，但是不超出其尺寸。这里注意一点， contain 是保持着 child 宽高比的大前提下，尽可能的填满，一般情况下，宽度或者高度达到最大值时，就会停止缩放。

![](/images/posts/flutter/2018-10-05-01.png)



**alignment:**
alignment ：对齐方式，默认的属性是 Alignment.center ，居中显示 child。

##### 源码

```
@override
RenderFittedBox createRenderObject(BuildContext context) {
return new RenderFittedBox(
  fit: fit,
  alignment: alignment,
  textDirection: Directionality.of(context),
);
}

```


FittedBox 具体实现是由 RenderFittedBox 进行的。目前的一些基础控件，继承自 RenderObjectWidget 的， widget 本身都只是存储了一些配置信息，真正的绘制渲染，则是由内部的 createRenderObject 所调用的 RenderObject 去实现的。

```

if (child != null) {
  child.layout(const BoxConstraints(), parentUsesSize: true);
  // 如果child不为null，则按照child的尺寸比率缩放child的尺寸
  size = constraints.constrainSizeAndAttemptToPreserveAspectRatio(child.size);
  _clearPaintData();
} else {
  // 如果child为null，则按照最小尺寸进行布局
  size = constraints.smallest;
}


```

###  AspectRatio

调整 child 的到设置的宽高比

#### AspectRatio 布局行为

- 1.AspectRatio 首先会在布局限制条件允许的范围内尽可能的扩展， widget 的高度是由宽度和比率决定的，类似于 BoxFit 中的 contain ，按照固定比率去尽量占满区域。
- 2.如果在满足所有限制条件过后无法找到一个可行的尺寸， AspectRatio 最终将会去优先适应布局限制条件，而忽略所设置的比率。

#### AspectRatio 继承关系

```
Object > Diagnosticable > DiagnosticableTree > Widget > RenderObjectWidget > SingleChildRenderObjectWidget > AspectRatio

```
所以 AspectRatio 也是一个基础控件

#### AspectRatio 示例

```
new Container(
  height: 200.0,
  child: new AspectRatio(
    aspectRatio: 1.5,
    child: new Container(
      color: Colors.red,
    ),
  ),
);

```
定义了一个高度为200的区域，内部 AspectRatio 比率设置为1.5，最终 AspectRatio 的是宽 300 高 200 的一个区域。

#### AspectRatio 源码解析

构造函数

```

const AspectRatio({
Key key,
@required this.aspectRatio,
Widget child
}) 

```

构造函数只包含了一个aspectRatio属性，其中aspectRatio不能为null。

##### 属性解析

`aspectRatio: `  aspectRatio 是宽高比，最终可能不会根据这个值去布局，具体则要看综合因素，外层是否允许按照这种比率进行布局，只是一个参考值。

##### 源码

```
@override
  RenderAspectRatio createRenderObject(BuildContext context) => new RenderAspectRatio(aspectRatio: aspectRatio);

```

绘制都是交由 RenderObject 去完成的，这里则是由 RenderAspectRatio 去完成具体的绘制工作。

RenderAspectRatio 的构造函数中会对 aspectRatio 做一些检测（assert）

- aspectRatio 不能为 null
- aspectRatio 必须大于 0 
- aspectRatio 必须是有限的

RenderAspectRatio  的具体尺寸计算函数：
1. 如果限制条件为 isTight ，则返回最小的尺寸（ constraints.smallest ）

```

if (constraints.isTight)
  return constraints.smallest;

```

2. 如果宽度为有限的值，则将高度设置为 width / _aspectRatio  。 如果宽度为无限，则将高度设为最大高度，宽度设为height * _aspectRatio ；

```
if (width.isFinite) {
  height = width / _aspectRatio;
} else {
  height = constraints.maxHeight;
  width = height * _aspectRatio;
}

```

3. 接下来则是在限制范围内调整宽高，总体思想则是宽度优先，大于最大值则设为最大值，小于最小值，则设为最小值。
3.1 如果宽度大于最大宽度，则将其设为最大宽度，高度设为 width / _aspectRatio ；

```
if (width > constraints.maxWidth) {
  width = constraints.maxWidth;
  height = width / _aspectRatio;
}

```

3.2 如果高度大于最大高度，则将其设为最大高度，宽度设为 height * _aspectRatio；

```
if (height > constraints.maxHeight) {
  height = constraints.maxHeight;
  width = height * _aspectRatio;
}
```

3.3 如果高度小于最小高度，则将其设为最小高度，宽度设为height * _aspectRatio。

```
if (height < constraints.minHeight) {
  height = constraints.minHeight;
  width = height * _aspectRatio;
}
```

#### AspectRatio 使用场景

AspectRatio 适用于需要固定宽高比的情景下。笔者最近使用这个控件的场景是相机，相机的预览尺寸都是固定的几个值，因此不能随意去设置相机的显示区域，必须按照比率进行显示，否则会出现拉伸的情况。除此之外，倒是用的不多。

### ConstrainedBox

这个控件的作用是添加额外的限制条件（constraints）到child上，本身挺简单的，可以被一些控件替换使用。

#### ConstrainedBox 布局行为

ConstrainedBox的布局行为非常简单，取决于设置的限制条件，

#### ConstrainedBox 继承关系

```
Object > Diagnosticable > DiagnosticableTree > Widget > RenderObjectWidget > SingleChildRenderObjectWidget > ConstrainedBox

```

所以 ConstrainedBox 也是一个基础控件

#### ConstrainedBox 示例

```

new ConstrainedBox(
  constraints: const BoxConstraints(
    minWidth: 100.0,
    minHeight: 100.0,
    maxWidth: 150.0,
    maxHeight: 150.0,
  ),
  child: new Container(
    width: 200.0,
    height: 200.0,
    color: Colors.red,
  ),
);

```

在一个宽高 200.0 的 Container 上添加一个约束最大最小宽高的 ConstrainedBox ，实际的显示中，则是一个宽高为 150.0 的区域。

#### ConstrainedBox 源码

构造函数

```

ConstrainedBox({
Key key,
@required this.constraints,
Widget child
})

```

包含一个 constraints ，但是不能为空。

##### 属性解析

`constraints ` ：添加到 child 上的额外限制条件，其类型为 BoxConstraints 。 BoxConstraints 的作用是限制各种最大最小宽高。并且 double.infinity 在 widget 布局的时候是合法的，也就说，例如想最大的扩展宽度，可以将宽度值设为 double.infinity。

##### 源码

```
@override
RenderConstrainedBox createRenderObject(BuildContext context) {
return new RenderConstrainedBox(additionalConstraints: constraints);
}

```

RenderConstrainedBox 实现其绘制。其具体的布局表现分两种情况：

- 1.如果 child 不为 null ，则将限制条件加在 child 上；
- 2.如果 child 为 null ，则会尽可能的缩小尺寸。

具体代码

```

@override
void performLayout() {
if (child != null) {
  child.layout(_additionalConstraints.enforce(constraints), parentUsesSize: true);
  size = child.size;
} else {
  size = _additionalConstraints.enforce(constraints).constrain(Size.zero);
}
}

```

#### ConstrainedBox 使用场景

需要添加额外的约束条件可以使用此控件，例如设置最小宽高，尽可能的占用区域等等

### UnconstrainedBox 
UnconstrainedBox 刚好与 ConstrainedBox 相反，是不添加任何约束条件到 child 上，让 child 按照其原始的尺寸渲染。