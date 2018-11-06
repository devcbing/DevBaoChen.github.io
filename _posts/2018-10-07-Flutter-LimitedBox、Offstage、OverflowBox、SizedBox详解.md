---
layout: post
title: LimitedBox、Offstage、OverflowBox、SizedBox详解
categories: [Flutter, ]
description: LimitedBox、Offstage、OverflowBox、SizedBox详解
keywords: Flutter, 
---

LimitedBox、Offstage、OverflowBox、SizedBox详解



![](/images/posts/flutter/2018-09-30-00.jpeg)

### LimitedBox

对最大宽高进行限制

#### LimitedBox 布局

LimitedBox 是将 child 限制在其最大宽高中，但是如果 LimitedBox 的最大宽高不受限制， child 就会受到这个最大宽高限制。

#### LimitedBox 继承关系

```
Object > Diagnosticable > DiagnosticableTree > Widget > RenderObjectWidget > SingleChildRenderObjectWidget > LimitedBox

```

#### LimitedBox 示例

```
Row(
  children: <Widget>[
    Container(
      color: Colors.red,
      width: 100.0,
    ),
    LimitedBox(
      maxWidth: 150.0,
      child: Container(
        color: Colors.blue,
        width: 250.0,
      ),
    ),
  ],
)

```

#### LimitedBox 源码

##### 构造函数

```

const LimitedBox({
  Key key,
  this.maxWidth = double.infinity,
  this.maxHeight = double.infinity,
  Widget child,
})

```

##### 属性解析

`maxWidth ` ：限定的最大宽度，默认值是 double.infinity ，不能为负数。
`maxHeight `：限定的最大高度，默认值是 double.infinity ，不能为负数。

##### 源码

LimitedBox 计算 constraint 的代码

```
minWidth: constraints.minWidth,
maxWidth: constraints.hasBoundedWidth ? constraints.maxWidth : constraints.constrainWidth(maxWidth),
minHeight: constraints.minHeight,
maxHeight: constraints.hasBoundedHeight ? constraints.maxHeight : constraints.constrainHeight(maxHeight)

```

布局代码

```
if (child != null) {
  child.layout(_limitConstraints(constraints), parentUsesSize: true);
  size = constraints.constrain(child.size);
} else {
  size = _limitConstraints(constraints).constrain(Size.zero);
}

```

根据最大尺寸，限制 child 的布局，然后将自身调节到 child 的尺寸。

### Offstage

通过一个参数，来控制 child 是否显示

#### Offstage 布局

Offstage  的布局行为完全取决于其 offstage 参数

- 1.当 offstage 为 true ，当前控件不会被绘制在屏幕上，不会响应点击事件，也不会占用空间；
- 2.当 offstage 为 false ，当前控件则跟平常用的控件一样渲染绘制

当 Offstage 不可见的时候，如果 child 有动画，应该手动停掉， Offstage 并不会停掉动画。

#### Offstage 继承关系

```

Object > Diagnosticable > DiagnosticableTree > Widget > RenderObjectWidget > SingleChildRenderObjectWidget > Offstage

```

#### Offstage 示例

```

Column(
  children: <Widget>[
    new Offstage(
      offstage: offstage,
      child: Container(color: Colors.blue, height: 100.0),
    ),
    new CupertinoButton(
      child: Text("点击切换显示"),
      onPressed: () {
        setState(() {
          offstage = !offstage;
        });
      },
    ),
  ],
)

```

当点击切换按钮的时候，可以看到 Offstage 显示消失。

#### Offstage 源码

##### 构造函数

```
const Offstage({ Key key, this.offstage = true, Widget child })

```

##### 属性解析

`offstage `：默认为 true ，也就是不显示，当为 false 的时候，会显示该控件。

##### 源码

 Offstage 的 computeIntrinsicSize 相关的方法：
```

@override
double computeMinIntrinsicWidth(double height) {
    if (offstage)
      return 0.0;
    return super.computeMinIntrinsicWidth(height);
  }

```

可以看到，当 offstage 为 true 的时候，自身的最小以及最大宽高都会被置为 0.0 。


hitTest方法：
```
  @override
  bool hitTest(HitTestResult result, { Offset position }) {
    return !offstage && super.hitTest(result, position: position);
  }

```

当 offstage 为 true 的时候，也不会去执行。



paint方法：
```
@override
  void paint(PaintingContext context, Offset offset) {
    if (offstage)
      return;
    super.paint(context, offset);
  }

```

当 offstage 为 true 的时候直接返回，不绘制了。

Offstage 并不是通过插入或者删除自己在 widget tree 中的节点，来达到显示以及隐藏的效果，而是通过设置自身尺寸、不响应 hitTest 以及不绘制，来达到展示与隐藏的效果。


#### Offstage 使用场景

当我们需要控制一个区域显示或者隐藏的时候，可以使用这个场景。其实也有其他代替的方法，但是成本会高很多，例如直接在tree上插入删除。


### OverflowBox

 OverflowBox 允许 child 超出 parent 的范围显示。

#### OverflowBox 布局

当 OverflowBox 的最大尺寸大于 child 的时候， child 可以完整显示，当其小于 child 的时候，则以最大尺寸为基准，当然，这个尺寸都是可以突破父节点的。最后加上对齐方式，完成布局。

#### OverflowBox 继承关系

```

Object > Diagnosticable > DiagnosticableTree > Widget > RenderObjectWidget > SingleChildRenderObjectWidget > OverflowBox

```

#### OverflowBox 示例

```

Container(
  color: Colors.green,
  width: 200.0,
  height: 200.0,
  padding: const EdgeInsets.all(5.0),
  child: OverflowBox(
    alignment: Alignment.topLeft,
    maxWidth: 300.0,
    maxHeight: 500.0,
    child: Container(
      color: Color(0x33FF00FF),
      width: 400.0,
      height: 400.0,
    ),
  ),
)

```

![](/images/posts/flutter/2018-10-07-01.png)

当 maxHeight 大于 height 的时候，可以完全显示下来，当 maxHeight 小于 height 的时候，则不会会被隐藏掉。


#### OverflowBox 源码

##### 构造函数

```

const OverflowBox({
    Key key,
    this.alignment = Alignment.center,
    this.minWidth,
    this.maxWidth,
    this.minHeight,
    this.maxHeight,
    Widget child,
  })

```

##### 属性解析

`alignment `：对齐方式。

`minWidth `：允许 child 的最小宽度。如果 child 宽度小于这个值，则按照最小宽度进行显示。

`maxWidth `：允许 child 的最大宽度。如果 child 宽度大于这个值，则按照最大宽度进行展示。

`minHeight `：允许 child 的最小高度。如果 child 高度小于这个值，则按照最小高度进行显示。

`maxHeight `：允许 child 的最大高度。如果 child 高度大于这个值，则按照最大高度进行展示。

其中，最小以及最大宽高度，如果为 null 的时候，就取父节点的 constraint 代替。


##### 源码

```
if (child != null) {
  child.layout(_getInnerConstraints(constraints), parentUsesSize: true);
  alignChild();
}

```
如果 child 不为 null , child 则会按照计算出的 constraints 进行尺寸的调整，然后对齐。

#### OverflowBox 使用场景

有时候设计图上出现的角标，会超出整个模块，可以使用 OverflowBox 控件。但我们应该知道，不使用这种控件，也可以完成布局，在最外面包一层，也能达到一样的效果。


### SizedBox

设置具体尺寸。

#### SizedBox 布局行为

- 1.child 不为 null 时，如果设置了宽高，则会强制把 child 尺寸调到此宽高；如果没有设置宽高，则会根据 child 尺寸进行调整；
- 2.child 为 null 时，如果设置了宽高，则自身尺寸调整到此宽高值，如果没设置，则尺寸为 0 ；

#### SizedBox 继承关系

```

Diagnosticable > DiagnosticableTree > Widget > RenderObjectWidget > SingleChildRenderObjectWidget > SizedBox

```

#### SizedBox 布局

```

Container(
  color: Colors.green,
  padding: const EdgeInsets.all(5.0),
  child: SizedBox(
    width: 200.0,
    height: 200.0,
    child: Container(
      color: Colors.red,
      width: 100.0,
      height: 300.0,
    ),
  ),
)

```


#### SizedBox 源码

##### 构造函数

```

const SizedBox({ Key key, this.width, this.height, Widget child })

```

##### 属性解析

`width `：宽度值，如果具体设置了，则强制 child 宽度为此值，如果没设置，则根据 child 宽度调整自身宽度。

`height `：高度值,如果具体设置了，则强制 child 高度为此值，如果没设置，则根据 child 宽度调整自身宽度。


##### 源码

```

SizedBox 内部是通过 RenderConstrainedBox 来实现的。具体的源码就不解析了，总体思路是，根据宽高值算好一个  constraints ，然后强制应用到 child 上。

```
#### SizedBox 使用场景

这个控件，很多场景可以使用。但是，可以替代它的控件也有不少，例如 Container 、 ConstrainedBox 等。而且 SizedBox 就是 ConstrainedBox 的一个特例。

