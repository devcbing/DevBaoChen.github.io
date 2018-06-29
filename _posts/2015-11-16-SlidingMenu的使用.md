---
layout: post
title: SlidingMenu的使用
categories: [Android, ]
description: SlidingMenu的使用
keywords: Android, 
---

SlidingMenu的使用


```
		SlidingMenu menu = new SlidingMenu(this);
		// 设置滑动方向
		menu.setMode(SlidingMenu.LEFT);
		// 设置监听开始滑动的触碰范围
		menu.setTouchModeAbove(SlidingMenu.TOUCHMODE_FULLSCREEN);
		// 设置边缘阴影的宽度，通过dimens资源文件中的ID设置
		menu.setShadowWidthRes(R.dimen.shadow_width);
		// 设置边缘阴影的颜色/图片，通过资源文件ID设置
		menu.setShadowDrawable(R.drawable.shadow);
		// 设置menu全部打开后，主界面剩余部分与屏幕边界的距离，通过dimens资源文件ID设置
		menu.setBehindOffsetRes(R.dimen.slidingmenu_offset);
		// 设置是否淡入淡出
		menu.setFadeEnabled(true);
		// 设置淡入淡出的值，只在setFadeEnabled设置为true时有效
		menu.setFadeDegree(0.35f);
		// 将menu绑定到Activity，同时设置绑定类型
		menu.attachToActivity(this, SlidingMenu.SLIDING_WINDOW);
		// 设置menu的layout
		menu.setMenu(R.layout.slide_menu);
		// 设置menu的背景颜色
		 menu.setBackgroundColor(getResources().getColor(
		 android.R.color.background_dark));
		// 获取menu的layout
		View menuroot = menu.getMenu();
		//设置menu布局中控件的事件
		Button button1 = (Button) menuroot.findViewById(R.id.Button1);
		button1.setOnClickListener(new OnClickListener() {

			@Override
			public void onClick(final View v) {
				// TODO Auto-generated method stub
Toast.makeText(MainActivity.this,"点击显示menu",Toast.LENGTH_LONG).show();
}});




```