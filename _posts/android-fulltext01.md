---
title: Android TextView全文
date: 2016-6-22 18:39:44
categories: Android
tags:
- Android
- 自定义View
---
<Excerpt in index | 首页摘要> 
>Android 自定义Textview（全文/收起）

<!-- more -->
<The rest of contents | 余下全文>  
简单说，项目需要长文章缩略显示5行，点击全文展开。  
  
能想到的实现方式有两种，  
1：继承TextView，重写测量和layout方法。  
2：利用Textview的一个属性，投机取巧实现  
  
目前比较赶项目，用第二种，将来时间宽裕了，在去实现第一种吧。  
（PS： 技术债务）  
  
简单介绍下，就是利用maxLines这个属性，平常时候显示5行，点击全文显示Integer.MAX_VALUE。  
  
<hr/>
上代码：自定义控件XML：  
```xml
	<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    android:layout_width="match_parent"
	    android:layout_height="wrap_content"
	    android:background="@color/alpha"
	    android:orientation="vertical">
	
	    <TextView
	        android:id="@+id/stretch_content"
	        android:layout_width="match_parent"
	        android:layout_height="wrap_content"
	        android:lineSpacingExtra="5dp"
	        android:background="@color/alpha"
	        android:textColor="@color/TvTextColor"
	        android:textSize="14sp" />
	
	    <TextView
	        android:id="@+id/stretch_fulltext"
	        android:layout_width="wrap_content"
	        android:layout_height="wrap_content"
	        android:layout_marginTop="10dp"
	        android:background="@color/alpha"
	        android:text="全文"
	        android:textColor="@color/selector_time_axis_green_gray"
	        android:textSize="14sp"
	        android:visibility="invisible" />
	</LinearLayout>  
```
  
具体实现代码：
```java
	package com.qingxiang.ui.view;
	
	import android.content.Context;
	import android.os.SystemClock;
	import android.util.AttributeSet;
	import android.view.View;
	import android.widget.FrameLayout;
	import android.widget.TextView;
	
	import com.qingxiang.ui.R;
	
	
	/**
	 * 全文TextView
	 *
	 * @author Gloomy
	 */
	public class StretchTextView extends FrameLayout implements View.OnClickListener {
	    public static final String TAG = "StretchTextView";
	
	    private TextView content;
	    private TextView fullText;
	    private boolean isFullText = false;
	    private int previewSize = 5;
	    private int lineCount;
	
	    public StretchTextView(Context context, AttributeSet attrs) {
	        super(context, attrs);
	        View view = View.inflate(context, R.layout.layout_stretch_textview, this);
	        content = (TextView) view.findViewById(R.id.stretch_content);
	        fullText = (TextView) view.findViewById(R.id.stretch_fulltext);
	        fullText.setOnClickListener(this);
	    }
	
	    public void setText(String text, boolean isFullText) {
	        content.setText(text);
	
	        this.isFullText = isFullText;
	        content.setMaxLines(StretchTextView.this.isFullText ? Integer.MAX_VALUE : previewSize);
	
	        fullText.setText(isFullText ? "收起" : "全文");
	        content.post(() -> {
	            lineCount = content.getLineCount();
	            if (lineCount <= previewSize)
	                fullText.setVisibility(View.INVISIBLE);
	            else
	                fullText.setVisibility(View.VISIBLE);
	        });
	    }
	
	
	    @Override
	    public void onClick(View v) {
	        if (fullText.getVisibility() != View.VISIBLE)
	            return;
	        isFullText = !isFullText;
	
	        if (listener != null)
	            listener.onChange(isFullText);
	
	        fullText.setText(isFullText ? "收起" : "全文");
	        content.setMaxLines(isFullText ? lineCount : previewSize);
	    }
	
	    onChangeLinstener listener;
	
	    public void setListener(onChangeLinstener listener) {
	        this.listener = listener;
	    }
	
	    public interface onChangeLinstener {
	        void onChange(boolean on);
	    }
	}
```