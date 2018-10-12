---
layout: post
title: ionic3 中软件盘将底部 tabs 向上顶 bug 解决办法
categories: [ionic, ]
description: ionic3 中软件盘将底部 tabs 向上顶 bug 解决办法
keywords: ionic, 
---

ionic3 中软件盘将底部 tabs 向上顶 bug 解决办法

### 问题描述
在 ionic3 中，当底部有 tabs 时，如果在 tabs 对应的 page 中存在输入操作，就会出现 底部 tabs 被软键盘向上顶的 bug。

### 解决办法

在 Tabs.ts 中, 加入下面的代码

```
import { Component, ElementRef, Renderer, ViewChild } from '@angular/core';
import { Events, Tabs } from 'ionic-angular';

import { AboutPage } from '../about/about';
import { ContactPage } from '../contact/contact';
import { HomePage } from '../home/home';

@Component({
  templateUrl: 'tabs.html'
})
export class TabsPage {
  @ViewChild('myTabs') tabRef: Tabs; //对应的加在 tabs.html 中 #myTabs
  mb: any;
  tab1Root = HomePage;
  tab2Root = AboutPage;
  tab3Root = ContactPage;

  constructor(private elementRef: ElementRef, private renderer: Renderer, private event: Events) {

  }
  ionViewDidLoad() {
    let tabs = this.queryElement(this.elementRef.nativeElement, '.tabbar');
    this.event.subscribe('hideTabs', () => {
      this.renderer.setElementStyle(tabs, 'display', 'none');
      let SelectTab = this.tabRef.getSelected()._elementRef.nativeElement;
      let content = this.queryElement(SelectTab, '.scroll-content');
      this.mb = content.style['margin-bottom'];
      this.renderer.setElementStyle(content, 'margin-bottom', '0')
    });
    this.event.subscribe('showTabs', () => {
      this.renderer.setElementStyle(tabs, 'display', '');
      let SelectTab = this.tabRef.getSelected()._elementRef.nativeElement;
      let content = this.queryElement(SelectTab, '.scroll-content');
      this.renderer.setElementStyle(content, 'margin-bottom', this.mb)
    })
  }
  queryElement(elem: HTMLElement, q: string): HTMLElement {
    return <HTMLElement>elem.querySelector(q);
  }
}
```
然后在相应的 page 中，加入下面的代码：

```

import { Component } from '@angular/core';
import { NavController, Events } from 'ionic-angular';

import { Keyboard } from '@ionic-native/keyboard';

@Component({
  selector: 'page-home',
  templateUrl: 'home.html'
})
export class HomePage {

  constructor(public navCtrl: NavController, private event: Events, private keyboard: Keyboard) {

  }
  ionViewDidLoad() {
    this.keyboard.onKeyboardShow().subscribe(() => this.event.publish('hideTabs'));
    this.keyboard.onKeyboardHide().subscribe(() => this.event.publish('showTabs'));
  }
}

```
就可以解决这个 bug

### 原理分析

主要原理就是 在 tabs 中用 event 做一个监听事件，用来控制隐藏或显示底部 tabs，并在相应的 page 界面进行注册监听。让键盘弹出时调用隐藏 tabs 事件，键盘收回时让 tabs 显示

参考：
[https://github.com/ionic-team/ionic/issues/7047](https://github.com/ionic-team/ionic/issues/7047)



