---
layout: post
title: 在dialog中显示gridview
categories: [Android, ]
description: 在dialog中显示gridview
keywords: Android, 
---

 平常我们看到的dialog都是列表样式的，有没有什么方式让dialog中的内容显示为九宫等样式呢？想到在dialog中显示gridview。
首先写一个dialog.xml

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#ffffff"
    android:orientation="vertical">

    <GridView
        android:id="@+id/gridview"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center"
        android:numColumns="4" />
</LinearLayout>

```
然后在java中用LayoutInflater将dialog.xml变为view。加入到dialog中。
```
 View view = LayoutInflater.from(MainActivity.this).inflate(R.layout.dialog, null);
        // 设置style 控制默认dialog带来的边距问题
        final Dialog dialog = new Dialog(this, R.style.common_dialog);
        dialog.setContentView(view);
        dialog.show();
```
这里的common_dialog是一个自己设置的dialog样式
```
 <!-- 默认的style -->
    <style name="common_dialog" parent="@android:style/Theme.Dialog">
        <item name="android:windowBackground">@android:color/transparent</item>
        <item name="android:windowNoTitle">true</item>
    </style>

```
然后向dialog中gridview中加入数据

```
 GridView gridview = (GridView) view.findViewById(R.id.gridview);
        final List<Map<String, Object>> item = getData();
        // SimpleAdapter对象，匹配ArrayList中的元素
        SimpleAdapter simpleAdapter = new SimpleAdapter(this, item, R.layout.gridview_item, new String[]{"itemName"}, new int[]{R.id.grid_name});
        gridview.setAdapter(simpleAdapter);

```
```
  /**
     * 将数据ArrayList中
     *
     * @return
     */
    private List<Map<String, Object>> getData() {
        List<Map<String, Object>> items = new ArrayList<Map<String, Object>>();
        String[] province = getResources().getStringArray(R.array.licence_province);
        for (int i = 0; i < province.length; i++) {
            Map<String, Object> item = new HashMap<String, Object>();
            item.put("itemName", province[i]);
            items.add(item);
        }
        return items;

    }
```



gridview的点击选择事件：
```
// 添加点击事件
        gridview.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> arg0, View arg1, int arg2, long arg3) {
                String[] provincess = getResources().getStringArray(R.array.licence_province);
                Toast.makeText(getApplicationContext(), "你按下了" + provincess[arg2], Toast.LENGTH_SHORT).show();
                text_view.setText(provincess[arg2]);
                dialog.dismiss();
            }
        });
```
### 下面是重点：
设置dialog的显示位置
```
 Window window = dialog.getWindow();
        window.getDecorView().setPadding(0, 0, 0, 0);
        WindowManager.LayoutParams params = window.getAttributes();
        params.width = WindowManager.LayoutParams.MATCH_PARENT;
        params.gravity = Gravity.CENTER;
        window.setAttributes(params);
```
我这里的数据是写在arrays.xml中的：
```
<?xml version="1.0" encoding="utf-8"?>
<resources>

    <string-array name="licence_province">
        <item>A</item>
        <item>B</item>
        <item>C</item>
        <item>D</item>
        <item>E</item>
        <item>F</item>
        <item>G</item>
        <item>H</item>
        <item>I</item>
        <item>J</item>
        <item>K</item>
        <item>L</item>
        <item>M</item>
        <item>N</item>
        <item>O</item>
        <item>P</item>
        <item>Q</item>
        <item>R</item>
        <item>S</item>
        <item>T</item>
        <item>U</item>
        <item>V</item>
        <item>W</item>
        <item>X</item>
        <item>Y</item>
        <item>Z</item>
    </string-array>

</resources>

```

### 下面是效果:

![效果.gif](http://upload-images.jianshu.io/upload_images/1365793-fd4e2c5dd825657a.gif?imageMogr2/auto-orient/strip)



















