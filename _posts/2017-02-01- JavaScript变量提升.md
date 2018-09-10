---
layout: post
title: JavaScript 变量提升
categories: [JavaScript, ]
description: JavaScript 变量提升
keywords: JavaScript, 
---

JavaScript 变量提升

JavaScript 引擎的工作方式是， 先解析代码，再逐行运行，在解析代码的过程中，首先会获取所有的被声明的变量；然后执行代码。这就会造成所有被声明的变量都会被提升到代码的头部，然后在执行代码。 这个过程就是 `代码提升(hoisting)`;

运行下面的代码会产生什么结果?
```
console.log(a);
var a = "我是变量";

```
按照我的的常规逻辑，代码先运行 `console.log(a);` ,但是这个时候 ，a 并没有被声明和赋值，所以会报错，但是实际上并不会出错。
运行结果：
上面的代码并不会报错：仍然正常运行，在控制台中会打印 `undefined`;
![](/images/posts/web/2017-02-01-01.png)

导致这个结果的原因正是 `变量提升`;
真正运行的是下面的代码：

```
var a;
console.log(a);
a = "我是变量";
```
但是这个代码运行的结果是 `undefined`, 表示变量a已声明，但还未赋值。
