---
layout: post
title: Container widget 详解
categories: [Flutter, ]
description: Container widget 详解
keywords: Flutter, 
---

Container widget 详解

![](/images/posts/flutter/2018-09-30-00.jpeg)


### 什么是 Container
Container 是一个结合了绘制（painting）、定位（positioning）以及尺寸（sizing）widget 的一个 组合 widget。

#### Container 的组成

- 1.最里层的是 child 元素;
- 2.child 被 外面的 padding 包裹着;
- 3.然后被额外的 constraints 限制着;
- 4.最后添加 margin ;

#### Container 的绘制过程

- 1.绘制 transform 效果;
- 2.绘制 decoration ;
- 3.绘制 child ;
- 4.最后绘制 foregroundDecoration ;

#### Container 调节自身尺寸

- 第一种情况 : Container 在没有子节点（children）的时候，会试图去变得足够大。除非 constraints 是 unbounded 限制，在这种情况下，Container 会试图去变得足够小。

- 第二种情况 : 带子节点的 Container ，会根据子节点尺寸调节自身尺寸，但是 Container 构造器中如果包含了 width 、 height 以及 constraints ，按照构造器中的参数来进行尺寸的调节。

#### Container 如何布局

一般顺序是：
-  对齐 (alignment)
- 调整自身尺寸适合子节点
- 使用 width、 height 和 constraints 进行布局
- 扩展自身去适应父节点
- 调整自身大小

也就是说：
- 如果没有子节点、没有设置 width 、 height 以及 constraints ，并且父节点没有设置 unbounded 的限制，Container 会将自身调整到足够小。
- 如果没有子节点、对齐方式（alignment），但是提供了 width 、 height 或者 constraints ，那么 Container 会根据自身以及父节点的限制，将自身调节到足够小。
- 如果没有子节点、 width 、 height 、 constraints 以及 alignment ，但是父节点提供了 bounded 限制，那么 Container 会按照父节点的限制，将自身调整到足够大
- 如果有 alignment ，父节点提供了 unbounded 限制，那么 Container 将会调节自身尺寸来包住 child
- 如果有 alignment ，并且父节点提供了 bounded 限制，那么 Container 会将自身调整的足够大（在父节点的范围内），然后将 child 根据 alignment 调整位置；
- 含有 child ，但是没有 width 、 height 、 constraints 以及 alignment ， Container 会将父节点的 constraints 传递给 child ，并且根据 child 调整自身。
- margin 以及 padding 属性也会影响到布局。

#### Container 继承关系

```
Object > Diagnosticable > DiagnosticableTree > Widget > StatelessWidget > Container
```
所以 Container 是一个 StatelessWidget ，并且内部还有许多的 widget。

#### Container 源码解析


##### Container 构造函数

```
Container({
    Key key,
    this.alignment,
    this.padding,
    Color color,
    Decoration decoration,
    this.foregroundDecoration,
    double width,
    double height,
    BoxConstraints constraints,
    this.margin,
    this.transform,
    this.child,
  })
```
`key` : Container 唯一标识符，用于查找更新。
`alignment` : 控制 child 的对齐方式，如果 container 或者 container 父节点尺寸大于 child 的尺寸，这个属性设置会起作用，有很多种对齐方式。
`padding` : decoration 内部的空白区域，如果有 child 的话， child 位于 padding 内部。 padding 与 margin 的不同之处在于， padding 是包含在 content 内，而 margin 则是外部边界，设置点击事件的话， padding 区域会响应，而 margin 区域不会响应。
`color`: 用来设置 container 背景色，如果 foregroundDecoration 设置的话，可能会遮盖 color 效果。

`decoration` ：绘制在 child 后面的装饰，设置了 decoration 的话，就不能设置 color 属性，否则会报错，此时应该在 decoration 中进行颜色的设置。

`foregroundDecoration` ：绘制在 child 前面的装饰。

`width` ：container 的宽度，设置为 double.infinity 可以强制在宽度上撑满，不设置，则根据 child 和父节点两者一起布局。

`height` ：container 的高度，设置为 double.infinity 可以强制在高度上撑满。

`constraints` ：添加到 child 上额外的约束条件。

`margin` ：围绕在 decoration 和 child 之外的空白区域，不属于内容区域。

`transform` ：设置 container 的变换矩阵，类型为 Matrix4 。

`child` ： container 中的内容 widget 。

##### Container 源码

```
decoration = decoration ?? (color != null ? new BoxDecoration(color: color) : null),

```
对于颜色的设置，最后都是转换为decoration来进行绘制的。如果同时包含decoration和color两种属性，则会报错。

```
@override
  Widget build(BuildContext context) {
    Widget current = child;

    if (child == null && (constraints == null || !constraints.isTight)) {
      current = new LimitedBox(
        maxWidth: 0.0,
        maxHeight: 0.0,
        child: new ConstrainedBox(constraints: const BoxConstraints.expand())
      );
    }

    if (alignment != null)
      current = new Align(alignment: alignment, child: current);

    final EdgeInsetsGeometry effectivePadding = _paddingIncludingDecoration;
    if (effectivePadding != null)
      current = new Padding(padding: effectivePadding, child: current);

    if (decoration != null)
      current = new DecoratedBox(decoration: decoration, child: current);

    if (foregroundDecoration != null) {
      current = new DecoratedBox(
        decoration: foregroundDecoration,
        position: DecorationPosition.foreground,
        child: current
      );
    }

    if (constraints != null)
      current = new ConstrainedBox(constraints: constraints, child: current);

    if (margin != null)
      current = new Padding(padding: margin, child: current);

    if (transform != null)
      current = new Transform(transform: transform, child: current);

    return current;
  }
```

Container 的 build 函数不长，绘制也是一个线性的判断的过程，一层一层的包裹着 widget ，去实现不同的样式。

最里层的是 child，如果为空或者其他约束条件，则最里层包含的为一个 LimitedBox ，然后依次是 Align 、 Padding 、 DecoratedBox 、前景 DecoratedBox 、 ConstrainedBox 、 Padding （实现 margin 效果）、Transform 。

Container 的源码本身并不复杂，复杂的是它的各种布局表现。我们谨记住一点，如果内部不设置约束，则按照父节点尽可能的扩大，如果内部有约束，则按照内部来。

#### Container 使用场景

- 需要设置间隔（这种情况下，如果只是单纯的间隔，也可以通过 Padding 来实现）；
- 需要设置背景色；
- 需要设置圆角或者边框的时候（ ClipRRect 也可以实现圆角效果）；
- 需要对齐（ Align 也可以实现）；
- 需要设置背景图片的时候（也可以使用 Stack 实现）。


### 参考：
[https://docs.flutter.io/flutter/widgets/Container-class.html](https://docs.flutter.io/flutter/widgets/Container-class.html)