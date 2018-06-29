---
layout: post
title: ListView中item与控件抢夺焦点解决办法
categories: [Android, ]
description: ListView中item与控件抢夺焦点解决办法
keywords: Android, 
---

ListView中item与控件抢夺焦点解决办法

- 1.将ListView中的Item布局中的子控件focusable属性设置为false
- 2.在getView方法中设置button.setFocusable(false)
- 3.设置item的根布局的属性android:descendantFocusability="blocksDescendants"

这三种方法都是为了让Button等控件不能获取焦点，从而使得item可以响应点击事件。

第三种方法使用起来比较方便

它是将item布局中的其他所有控件都设置为不能获取焦点

- android:descendantFocusability属性共有三个取值，分别为
- beforeDescendants：viewgroup会优先其子类控件而获取到焦点
- afterDescendants：viewgroup 只有当其子类控件不需要获取焦点时才获取焦点
- blocksDescendants：viewgroup 会覆盖子类控件而直接获得焦点
