---
layout: post
title: RecyclerView 实现列表倒叙显示
categories: [Android, RecyclerView]
description: RecyclerView 实现列表倒叙显示
keywords: Android, RecyclerView
---

RecyclerView 实现列表倒叙显示

### 描述:

我们在使用 android 实现列表的时候，往往最新的数据会显示在最后一条，也就是页面底部, 但是有些需求是需要我们把最新的数据显示在列表最前面，那么如何实现呢，当我们实现 RecyclerView 来实现列表的时候，可以非常简单的实现这个需求。

### 实现方式:

#### 一、
将数据反转，然后填充到列表中

#### 二、

使用下面的代码，将整个列表进行反转

```
   //创建布局管理
    LinearLayoutManager layoutManager = new LinearLayoutManager(this);
    layoutManager.setStackFromEnd(true);//列表再底部开始展示，反转后由上面开始展示
    layoutManager.setReverseLayout(true);//列表翻转
    layoutManager.setOrientation(LinearLayoutManager.VERTICAL);//方向
    recyclerView.addItemDecoration(new DividerItemDecoration(this, DividerItemDecoration.VERTICAL)); //添加分割线

```
上面中要的是 `setStackFromEnd(true)  和 setReverseLayout(true)` 
这个方法， 就是使得我们的数据最新的在第一条

### 补充: 
如果使用 ListView ，则需要将最数据倒叙加载在列表中，并且有个问题：显示最后一条需要每次都移动到相应位置.

参考:[https://www.jianshu.com/p/2e647bf1573e](https://www.jianshu.com/p/2e647bf1573e)
