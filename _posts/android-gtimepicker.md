---
title: Android3D滚轮时间选择器
date: 2016-9-22 18:31:02
categories: Android
tags: 
- Android
- 自定义View
---
<Excerpt in index | 首页摘要> 
>Android3D滚轮时间选择器
<!-- more -->
<The rest of contents | 余下全文> 
  
[GTimePicker Github下载地址](https://github.com/Gloomyer/GTimePicker)
  
##  效果图  ##
![](http://gloomyer.com/img/img/gtimepicker_demo.gif)
##  概述  ##
这个项目是基于[PickerView](https://github.com/saiwu-bigkoo/Android-PickerView)的一次二次封装！！！
  
老的项目有点庞大了，我这边只需要一个小时和分钟的选择.  
  
所以通过对[PickerView](https://github.com/saiwu-bigkoo/Android-PickerView)的二次封装来实现.  
  
##  使用说明  ##
下载库,导入进你的项目.里面有demo.  
  
布局:
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <com.gloomyer.gtimepicker.GTimePIcker
        android:id="@+id/gtp"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />

    <TextView
        android:id="@+id/tv"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="时间"
        android:textSize="18sp" />

</LinearLayout>
```
  
具体代码:
```java
package com.gloomyer.demo;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.Toast;

import com.gloomyer.gtimepicker.GTimePIcker;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        final GTimePIcker gtp = (GTimePIcker) findViewById(R.id.gtp);
        //设置初始显示时间
        gtp.setTime(5, 59);
        findViewById(R.id.tv).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                //获取当前选中的时间
                Toast.makeText(MainActivity.this, gtp.getTime(), Toast.LENGTH_LONG).show();
            }
        });
    }
}

```
  
  
##  样式修改说明  ##
在GTimePicker中
src/main/res/values/colors.xml中
修改:
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="pickerview_wheelview_textcolor_out">#a0a0a0</color>
    <color name="pickerview_wheelview_textcolor_center">#78d2a0</color>
    <color name="pickerview_wheelview_textcolor_divider">#78d2a0</color>
</resources>

```
  
pickerview_wheelview_textcolor_out:没有选中的文字颜色  
pickerview_wheelview_textcolor_center:被选中的文字颜色  
pickerview_wheelview_textcolor_divider:切割线的分隔颜色

字体大小在:
src/main/res/values/dimensions.xml中:
  
修改textSize
  
[详情移步至GTimePicker](https://github.com/Gloomyer/GTimePicker)